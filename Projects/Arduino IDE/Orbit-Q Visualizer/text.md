# Orbit-Q Visualizer V2: Build & Code Walkthrough

*This tutorial explains `Orbit-Q_Visualizer_v2.ino`: a 6-mode ambient display running on the ORBIT-Q board with an STM32F405RGT6 MCU card, driving a 128×32 I2C OLED and a 10-pixel WS2812B strip as one coordinated visual surface, not two separate animations. If you're following along to build this yourself, read the "Core Concept" section first — every mode in the code only makes sense once you understand that one idea.*

### 1. What this demo actually does
Six modes cycle automatically, ~90 seconds total, then loop forever:

| # | Mode | OLED shows | LED strip shows
| --- | --- | --- | --- |
| 0 | Wave Cross | scrolling waveform | the same waveform function, sampled past the right edge of the screen |
| 1 | Orbit Sweep | a dot orbiting with a fading trail | a comet at the matching angular position |
| 2 | Plasma Bloom | a dithered 2D plasma field | the same field, sampled off-screen |
| 3 | Radar Sweep | rotating sweep line + 4 decaying blips | a sharp point mirroring the sweep angle, flashing on blip hits |
| 4 | VU Spectrum | 8 bars driven by synthesized envelopes (not real audio — there's no mic on this board) | a level-meter bar with a decaying peak-hold marker |
| 5 | Matrix Cascade | falling column drops | a flash on the mapped LED the instant a column's drop crosses the bottom edge |

Transitions fade in/out over 600 ms so mode changes aren't an abrupt cut.

### 2. Hardware you need
- ORBIT-Q board with the STM32F405RGT6 MCU card (Cortex-M4F core, hardware single-precision FPU)
- SSD1306 128×32 OLED, I2C, address `0x3C`.
- WS2812B strip/ring, 10 pixels
- ST-Link (or your usual ORBIT-Q upload path)

### Wiring used in this sketch
- OLED → I2C1 default pins (SDA/SCL), power from on-board power supply.
- WS2812B data - `PB3`
- Onboard status LED - `PC13`

*These pins are already brought out on the ORBIT-Q carrier to the onboard OLED, LED strip header, and status LED — no extra wiring needed beyond confirming your card's pin definitions match.*

### 3. Libraries (Arduino IDE → Library Manager)
- `Adafruit GFX Library`
- `Adafruit SSD1306`
- `Adafruit NeoPixel`
- The two fonts (`FreeSansBold9pt7b`, `FreeSans9pt7b`) ship inside Adafruit GFX — no separate install.
- STM32 board support package installed via Boards Manager (`STM32duino`), with the F405RGT6 variant selected.
---

### CODE

`
cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_NeoPixel.h>
#include <Fonts/FreeSansBold9pt7b.h>
#include <Fonts/FreeSans9pt7b.h>
#include <math.h>

/* =========================================================
   CONCEPT v2
   ---------------------------------------------------------
   Same core idea as the original: the OLED and the LED strip
   are not two independent animations. Every mode is a
   function of one shared "phase" clock (t), and wherever
   possible the LED strip is treated as a physical extension
   of the OLED's coordinate space rather than a separate
   effect — e.g. the plasma field samples the identical
   plasmaField(x,y,t) that draws the screen, just at x values
   past the right edge; the matrix mode's LED flashes fire at
   the exact moment a column's drop falls off the bottom of
   the visible screen.

   WHAT CHANGED FROM v1:
   - 6 modes instead of 2 (wave cross, orbit sweep, plasma
     bloom, radar sweep w/ blips, VU-style spectrum bars,
     matrix-style cascade), each with its own visual identity
     but the same shared-field philosophy.
   - All sinf()/cosf() calls replaced with an 8-bit sine LUT.
     The STM32F103 ("blue pill") has no hardware FPU, so
     runtime sinf/cosf are software-emulated and comparatively
     slow; a 256-entry lookup table is the standard embedded
     fix and is what makes 6 richer modes affordable inside
     the same 20 ms frame budget.
   - Per-mode duration array instead of one fixed duration, so
     pacing can be tuned per effect (a full cycle through all
     6 modes is now roughly 90 seconds, then it loops
     indefinitely — this is meant to run continuously as a
     demo/kiosk show, not a one-shot animation).
   - Per-mode state (radar blip decay, VU peak-hold, matrix
     column phase) is reset on mode entry so switching modes
     never carries stale state into the next effect.

   HONESTY NOTE: the "VU spectrum" mode is a synthesized
   envelope (sums of sine harmonics), not a real audio input —
   there is no microphone in this design. It's labeled that way
   in code so it's not misrepresented as audio-reactive.

   Everything remains non-blocking (millis-paced) outside of
   the one-time boot animation, exactly as in v1.
   ========================================================= */

/* =======================
   MCU / BOARD DEFINES
   ======================= */
#define PC13_LED    PC13

/* =======================
   WS2812B CONFIG
   ======================= */
#define WS_PIN      PB3
#define NUMPIXELS   10
#define MAX_BRIGHTNESS  60   // ceiling brightness for the show (0-255)

Adafruit_NeoPixel pixels(NUMPIXELS, WS_PIN, NEO_GRB + NEO_KHZ800);

/* =======================
   OLED CONFIG (I2C1 default)
   ======================= */
#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT 32
#define OLED_RESET    -1
#define OLED_ADDR     0x3C

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

/* =======================
   TIMING / SHOW STATE
   ======================= */
const uint16_t FRAME_MS      = 20;     // ~50 fps render pace
const uint16_t TRANSITION_MS = 600;    // fade in/out window at mode edges
const uint8_t  NUM_MODES     = 6;

// per-mode runtime; sum ≈ 94 s per full cycle, then it repeats forever
const uint32_t MODE_DURATION[NUM_MODES] = {
  16000, // 0 wave cross
  15000, // 1 orbit sweep
  17000, // 2 plasma bloom
  15000, // 3 radar sweep
  17000, // 4 VU spectrum (simulated)
  14000  // 5 matrix cascade
};

unsigned long frameMillis   = 0;
unsigned long modeStartMs   = 0;
unsigned long pc13Millis    = 0;
uint8_t  currentMode        = 0;
bool     pc13State          = false;

float phase = 0.0f;
const float PHASE_STEP = 0.045f;   // tune this to speed up/slow down the whole show

/* =======================
   8-BIT SINE LOOKUP TABLE
   sin(2*pi*i/256) * 127, i = 0..255.
   Regular flash-resident const array — fine as-is on STM32
   (and ESP32/SAMD); if this is ever ported to classic AVR,
   wrap it in PROGMEM and read with pgm_read_byte().
   ======================= */
const int8_t SINE_TABLE[256] = {
     0,    3,    6,    9,   12,   16,   19,   22,   25,   28,   31,   34,   37,   40,   43,   46,
    49,   51,   54,   57,   60,   63,   65,   68,   71,   73,   76,   78,   81,   83,   85,   88,
    90,   92,   94,   96,   98,  100,  102,  104,  106,  107,  109,  111,  112,  113,  115,  116,
   117,  118,  120,  121,  122,  122,  123,  124,  125,  125,  126,  126,  126,  127,  127,  127,
   127,  127,  127,  127,  126,  126,  126,  125,  125,  124,  123,  122,  122,  121,  120,  118,
   117,  116,  115,  113,  112,  111,  109,  107,  106,  104,  102,  100,   98,   96,   94,   92,
    90,   88,   85,   83,   81,   78,   76,   73,   71,   68,   65,   63,   60,   57,   54,   51,
    49,   46,   43,   40,   37,   34,   31,   28,   25,   22,   19,   16,   12,    9,    6,    3,
     0,   -3,   -6,   -9,  -12,  -16,  -19,  -22,  -25,  -28,  -31,  -34,  -37,  -40,  -43,  -46,
   -49,  -51,  -54,  -57,  -60,  -63,  -65,  -68,  -71,  -73,  -76,  -78,  -81,  -83,  -85,  -88,
   -90,  -92,  -94,  -96,  -98, -100, -102, -104, -106, -107, -109, -111, -112, -113, -115, -116,
  -117, -118, -120, -121, -122, -122, -123, -124, -125, -125, -126, -126, -126, -127, -127, -127,
  -127, -127, -127, -127, -126, -126, -126, -125, -125, -124, -123, -122, -122, -121, -120, -118,
  -117, -116, -115, -113, -112, -111, -109, -107, -106, -104, -102, -100,  -98,  -96,  -94,  -92,
   -90,  -88,  -85,  -83,  -81,  -78,  -76,  -73,  -71,  -68,  -65,  -63,  -60,  -57,  -54,  -51,
   -49,  -46,  -43,  -40,  -37,  -34,  -31,  -28,  -25,  -22,  -19,  -16,  -12,   -9,   -6,   -3,
};

inline uint8_t angleIdx(float radians) {
  float w = fmodf(radians, 2.0f * PI);
  if (w < 0) w += 2.0f * PI;
  return (uint8_t)(w * (256.0f / (2.0f * PI)));
}
inline float sinTab(uint8_t idx) { return SINE_TABLE[idx] / 127.0f; }
inline float cosTab(uint8_t idx) { return SINE_TABLE[(uint8_t)(idx + 64)] / 127.0f; }
inline float fastSinF(float radians) { return sinTab(angleIdx(radians)); }
inline float fastCosF(float radians) { return cosTab(angleIdx(radians)); }
inline float wrapAngle(float a) {
  a = fmodf(a, 2.0f * PI);
  if (a < 0) a += 2.0f * PI;
  return a;
}

/* =======================
   FUNCTION DECLARATIONS
   ======================= */
void blinkPC13();
void ntxBootAnimation();
void updateShow();
void resetModeState(uint8_t mode);
float waveAt(float x, float t);
float plasmaField(float x, float y, float t);
float barEnvelope(int i, float t);
void renderWaveCross(float t, float fade);
void renderOrbitSweep(float t, float fade);
void renderPlasmaBloom(float t, float fade);
void renderRadarSweep(float t, float fade);
void renderVuSpectrum(float t, float fade);
void renderMatrixCascade(float t, float fade);

/* =======================
   MODE 3 STATE: radar blips
   ======================= */
struct Blip { float angle; float radiusFrac; float brightness; };
Blip blips[4] = {
  {0.70f, 0.85f, 0.0f},
  {2.30f, 0.55f, 0.0f},
  {4.10f, 0.95f, 0.0f},
  {5.40f, 0.35f, 0.0f}
};

/* =======================
   MODE 4 STATE: VU peak hold
   ======================= */
float peakLevel = 0.0f;

/* =======================
   MODE 5 STATE: matrix columns
   ======================= */
const int NUM_COLS = 16;
float prevDropPos[NUM_COLS];
float flashBrightness[NUMPIXELS];

/* =======================
   SETUP
   ======================= */
void setup() {
  pinMode(PC13_LED, OUTPUT);
  digitalWrite(PC13_LED, HIGH); // OFF (active low)

  pinMode(WS_PIN, OUTPUT);
  pixels.begin();
  pixels.setBrightness(MAX_BRIGHTNESS);
  pixels.clear();
  pixels.show();

  delay(300); // OLED power-up delay

  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    while (1);
  }

  delay(100);
  ntxBootAnimation();

  resetModeState(currentMode);
  frameMillis = millis();
  modeStartMs = millis();
}

/* =======================
   LOOP
   ======================= */
void loop() {
  blinkPC13();
  updateShow();
}

/* =======================
   PC13 HEARTBEAT
   ======================= */
void blinkPC13() {
  if (millis() - pc13Millis >= 500) {
    pc13Millis = millis();
    pc13State = !pc13State;
    digitalWrite(PC13_LED, pc13State ? LOW : HIGH);
  }
}

/* =======================
   MODE 0: WAVE CROSS
   OLED draws a scrolling waveform. The LED strip is treated as
   pixels sitting just past x=127, so it displays the SAME wave
   function continuing off the edge of the screen.
   ======================= */
float waveAt(float x, float t) {
  return 0.6f * fastSinF(x * 0.055f + t) + 0.4f * fastSinF(x * 0.13f - t * 1.35f);
}

void renderWaveCross(float t, float fade) {
  display.clearDisplay();

  int prevY = -1;
  for (int x = 0; x < SCREEN_WIDTH; x++) {
    float v = waveAt((float)x, t);
    int y = (SCREEN_HEIGHT / 2) + (int)(v * (SCREEN_HEIGHT / 2 - 3));
    if (prevY >= 0) display.drawLine(x - 1, prevY, x, y, SSD1306_WHITE);
    prevY = y;
  }
  display.display();

  for (int i = 0; i < NUMPIXELS; i++) {
    float ledX = SCREEN_WIDTH + i * 7.0f;
    float v = waveAt(ledX, t);
    float amp = (v + 1.0f) * 0.5f;

    uint16_t hue = (uint16_t)(30000 + amp * 20000); // blue -> violet -> pink band
    uint8_t val  = (uint8_t)(amp * 255);
    pixels.setPixelColor(i, pixels.gamma32(pixels.ColorHSV(hue, 255, val)));
  }
  pixels.setBrightness((uint8_t)(MAX_BRIGHTNESS * fade));
  pixels.show();
}

/* =======================
   MODE 1: ORBIT SWEEP
   One rotating angle drives a dot orbiting on the OLED (with a
   short fading trail) AND a comet sweep on the LED ring at the
   same angular position.
   ======================= */
void renderOrbitSweep(float t, float fade) {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(43, 3);
  display.print("ORBIT-Q");

  const int cx = 64, cy = 20, rx = 50, ry = 9;
  for (int trail = 5; trail >= 0; trail--) {
    float a = t - trail * 0.15f;
    int dx = cx + (int)(rx * fastCosF(a));
    int dy = cy + (int)(ry * fastSinF(a));
    if (trail == 0) display.fillCircle(dx, dy, 2, SSD1306_WHITE);
    else            display.drawPixel(dx, dy, SSD1306_WHITE);
  }
  display.display();

  float angleNorm = wrapAngle(t);
  float pos = (angleNorm / (2.0f * PI)) * NUMPIXELS;
  uint16_t baseHue = (uint16_t)(angleNorm / (2.0f * PI) * 65535.0f);

  for (int i = 0; i < NUMPIXELS; i++) {
    float d = fabsf((float)i - pos);
    d = fminf(d, NUMPIXELS - d);
    float b = fmaxf(0.0f, 1.0f - d / 2.2f);

    uint8_t val = (uint8_t)(b * 255);
    pixels.setPixelColor(i, pixels.gamma32(pixels.ColorHSV(baseHue, 255, val)));
  }
  pixels.setBrightness((uint8_t)(MAX_BRIGHTNESS * fade));
  pixels.show();
}

/* =======================
   MODE 2: PLASMA BLOOM
   Classic three-harmonic plasma field, rendered on the OLED at
   quarter resolution (32x8 grid, blitted as 4x4 blocks) with a
   two-level dither to fake a mid-tone on a 1-bit display. The
   LED strip samples the exact same plasmaField() at x positions
   past the screen edge - identical mechanism to Mode 0's "shared
   coordinate space", just with a richer 2D field.
   ======================= */
float plasmaField(float x, float y, float t) {
  return fastSinF(x * 0.10f + t)
       + fastSinF(y * 0.18f - t * 0.8f)
       + fastSinF((x + y) * 0.05f + t * 1.3f);   // range approx -3..3
}

void renderPlasmaBloom(float t, float fade) {
  display.clearDisplay();
  const int gridCols = 32, gridRows = 8;
  const int bw = SCREEN_WIDTH / gridCols;   // 4
  const int bh = SCREEN_HEIGHT / gridRows;  // 4

  for (int gy = 0; gy < gridRows; gy++) {
    for (int gx = 0; gx < gridCols; gx++) {
      float px = gx * bw + bw * 0.5f;
      float py = gy * bh + bh * 0.5f;
      float n = plasmaField(px, py, t) / 3.0f;   // -1..1

      if (n > 0.35f) {
        display.fillRect(gx * bw, gy * bh, bw, bh, SSD1306_WHITE);       // bright block
      } else if (n > -0.15f) {
        display.fillRect(gx * bw + 1, gy * bh + 1, bw - 2, bh - 2, SSD1306_WHITE); // mid-tone dot
      }
    }
  }
  display.display();

  for (int i = 0; i < NUMPIXELS; i++) {
    float ledX = SCREEN_WIDTH + i * 9.0f;
    float n = constrain(plasmaField(ledX, SCREEN_HEIGHT * 0.5f, t) / 3.0f, -1.0f, 1.0f);
    float amp = (n + 1.0f) * 0.5f;

    uint16_t hue = (uint16_t)(38000 + amp * 10000); // teal -> green -> lime, distinct from Mode 0's palette
    uint8_t val  = (uint8_t)(amp * 255);
    pixels.setPixelColor(i, pixels.gamma32(pixels.ColorHSV(hue, 255, val)));
  }
  pixels.setBrightness((uint8_t)(MAX_BRIGHTNESS * fade));
  pixels.show();
}

/* =======================
   MODE 3: RADAR SWEEP
   A sweep line rotates around a dotted elliptical boundary. Four
   fixed "targets" flash and decay when the sweep passes their
   angle. The LED strip mirrors the sweep as a crisp point (not a
   soft comet, to read as a radar blip) and overlays a white flash
   on whichever LED corresponds to a target's angle while it's lit.
   ======================= */
void renderRadarSweep(float t, float fade) {
  display.clearDisplay();
  const int cx = 64, cy = 16;
  const float rx = 58.0f, ry = 13.0f;

  for (uint8_t s = 0; s < 252; s += 6) {
    int fx = cx + (int)(rx * cosTab(s));
    int fy = cy + (int)(ry * sinTab(s));
    display.drawPixel(fx, fy, SSD1306_WHITE);
  }

  float a = wrapAngle(t * 0.6f);
  int ex = cx + (int)(rx * fastCosF(a));
  int ey = cy + (int)(ry * fastSinF(a));
  display.drawLine(cx, cy, ex, ey, SSD1306_WHITE);

  for (int i = 0; i < 4; i++) {
    float d = fabsf(wrapAngle(a - blips[i].angle));
    if (d > PI) d = 2.0f * PI - d;
    if (d < 0.12f) blips[i].brightness = 1.0f;      // sweep just crossed this target
    else           blips[i].brightness *= 0.93f;     // decay (~20 ms/frame)

    int bx = cx + (int)(blips[i].radiusFrac * rx * fastCosF(blips[i].angle));
    int by = cy + (int)(blips[i].radiusFrac * ry * fastSinF(blips[i].angle));
    if (blips[i].brightness > 0.15f) display.fillCircle(bx, by, 2, SSD1306_WHITE);
    else                             display.drawPixel(bx, by, SSD1306_WHITE);
  }
  display.display();

  float pos = (a / (2.0f * PI)) * NUMPIXELS;
  for (int i = 0; i < NUMPIXELS; i++) {
    float d = fabsf((float)i - pos);
    d = fminf(d, NUMPIXELS - d);
    float sweepB = fmaxf(0.0f, 1.0f - d / 1.3f);      // sharper falloff than Mode 1's comet
    uint32_t c = pixels.gamma32(pixels.ColorHSV(24000, 255, (uint8_t)(sweepB * 200)));

    for (int b = 0; b < 4; b++) {
      if (blips[b].brightness > 0.05f) {
        float bpos = (blips[b].angle / (2.0f * PI)) * NUMPIXELS;
        float bd = fabsf((float)i - bpos);
        bd = fminf(bd, NUMPIXELS - bd);
        if (bd < 1.0f) c = pixels.gamma32(pixels.ColorHSV(0, 0, (uint8_t)(blips[b].brightness * 255)));
      }
    }
    pixels.setPixelColor(i, c);
  }
  pixels.setBrightness((uint8_t)(MAX_BRIGHTNESS * fade));
  pixels.show();
}

/* =======================
   MODE 4: VU SPECTRUM (simulated envelope, not real audio)
   8 bars, each driven by its own sum of two sine harmonics so
   they move independently rather than in lockstep. The LED strip
   acts as a level meter of the averaged "loudness" with a
   decaying peak-hold marker - a standard VU meter behavior.
   ======================= */
float barEnvelope(int i, float t) {
  float base   = 0.5f + 0.5f * fastSinF(t * (0.9f + 0.17f * i) + i * 0.85f);
  float jitter = 0.5f + 0.5f * fastSinF(t * (3.1f + 0.4f * i) + i * 2.3f);
  return base * 0.7f + jitter * 0.3f;   // 0..1
}

void renderVuSpectrum(float t, float fade) {
  display.clearDisplay();
  const int NUM_BARS = 8;
  const int barW = 12, gap = 4;
  const int totalW = NUM_BARS * barW + (NUM_BARS - 1) * gap;
  int startX = (SCREEN_WIDTH - totalW) / 2;
  float levelSum = 0.0f;

  for (int i = 0; i < NUM_BARS; i++) {
    float env = barEnvelope(i, t);
    levelSum += env;
    int h = (int)(env * (SCREEN_HEIGHT - 4));
    int x = startX + i * (barW + gap);
    int y = SCREEN_HEIGHT - 1 - h;
    display.fillRect(x, y, barW, h, SSD1306_WHITE);
  }
  display.display();

  float level = levelSum / NUM_BARS;
  peakLevel = fmaxf(level, peakLevel * 0.985f);

  int lit = (int)(level * NUMPIXELS);
  for (int i = 0; i < NUMPIXELS; i++) {
    if (i < lit) {
      float frac = (float)i / (NUMPIXELS - 1);
      uint16_t hue = (uint16_t)(25000 - frac * 25000);   // green -> red along the strip
      pixels.setPixelColor(i, pixels.gamma32(pixels.ColorHSV(hue, 255, 255)));
    } else {
      pixels.setPixelColor(i, 0);
    }
  }
  int peakIdx = constrain((int)(peakLevel * NUMPIXELS), 0, NUMPIXELS - 1);
  pixels.setPixelColor(peakIdx, pixels.Color(255, 255, 255));   // peak-hold marker

  pixels.setBrightness((uint8_t)(MAX_BRIGHTNESS * fade));
  pixels.show();
}

/* =======================
   MODE 5: MATRIX CASCADE
   Falling column drops, purely integer/positional (no trig) for
   variety in technique. Each column's vertical position is a
   direct function of the shared phase (fmodf), so no per-frame
   integration is needed. When a column's drop crosses the bottom
   of the OLED, the mapped LED gets a bright flash - the drop
   "falling off the screen" and continuing on the strip, the same
   metaphor as Mode 0.
   ======================= */
void resetModeState(uint8_t mode) {
  switch (mode) {
    case 3:
      for (int i = 0; i < 4; i++) blips[i].brightness = 0.0f;
      break;
    case 4:
      peakLevel = 0.0f;
      break;
    case 5:
      for (int i = 0; i < NUM_COLS; i++) {
        float speed  = 1.4f + (i % 5) * 0.35f;
        float offset = i * 17.3f;
        prevDropPos[i] = fmodf(phase * speed * 6.0f + offset, SCREEN_HEIGHT + 10.0f);
      }
      for (int i = 0; i < NUMPIXELS; i++) flashBrightness[i] = 0.0f;
      break;
  }
}

void renderMatrixCascade(float t, float fade) {
  display.clearDisplay();
  const int colW = SCREEN_WIDTH / NUM_COLS;   // 8
  const int trailLen = 10;
  const float cycleLen = SCREEN_HEIGHT + trailLen;

  for (int i = 0; i < NUM_COLS; i++) {
    float speed  = 1.4f + (i % 5) * 0.35f;
    float offset = i * 17.3f;
    float dropPos = fmodf(t * speed * 6.0f + offset, cycleLen);

    if (dropPos < prevDropPos[i]) {                      // wrapped -> fell off the bottom
      int ledIdx = (i * NUMPIXELS) / NUM_COLS;
      flashBrightness[ledIdx] = 1.0f;
    }
    prevDropPos[i] = dropPos;

    int headY = (int)dropPos;
    int x = i * colW + colW / 2;
    display.drawFastVLine(x, headY - 2, 3, SSD1306_WHITE);
    for (int k = 4; k < trailLen; k += 3) {
      int ty = headY - k;
      if (ty >= 0 && ty < SCREEN_HEIGHT) display.drawPixel(x, ty, SSD1306_WHITE);
    }
  }
  display.display();

  for (int i = 0; i < NUMPIXELS; i++) {
    float ambient = 0.10f + 0.05f * fastSinF(t * 0.5f + i * 0.7f);
    float b = fmaxf(ambient, flashBrightness[i]);
    pixels.setPixelColor(i, pixels.gamma32(pixels.ColorHSV(27000, 255, (uint8_t)(constrain(b, 0.0f, 1.0f) * 255))));
    flashBrightness[i] *= 0.80f;
  }
  pixels.setBrightness((uint8_t)(MAX_BRIGHTNESS * fade));
  pixels.show();
}

/* =======================
   SHOW SEQUENCER
   Advances the shared phase clock once per frame, handles mode
   switching using each mode's own duration, resets mode-local
   state on entry, and computes a fade multiplier at each mode's
   edges so transitions aren't an abrupt cut.
   ======================= */
void updateShow() {
  unsigned long now = millis();
  if (now - frameMillis < FRAME_MS) return;
  frameMillis = now;

  unsigned long elapsed = now - modeStartMs;
  if (elapsed >= MODE_DURATION[currentMode]) {
    currentMode = (currentMode + 1) % NUM_MODES;
    modeStartMs = now;
    elapsed = 0;
    resetModeState(currentMode);
  }

  float fade = 1.0f;
  uint32_t dur = MODE_DURATION[currentMode];
  if (elapsed < TRANSITION_MS) {
    fade = (float)elapsed / TRANSITION_MS;
  } else if (elapsed > dur - TRANSITION_MS) {
    fade = (float)(dur - elapsed) / TRANSITION_MS;
  }
  fade = constrain(fade, 0.0f, 1.0f);

  switch (currentMode) {
    case 0: renderWaveCross(phase, fade);     break;
    case 1: renderOrbitSweep(phase, fade);    break;
    case 2: renderPlasmaBloom(phase, fade);   break;
    case 3: renderRadarSweep(phase, fade);    break;
    case 4: renderVuSpectrum(phase, fade);    break;
    case 5: renderMatrixCascade(phase, fade); break;
  }

  phase += PHASE_STEP;
}

/* =======================
   NTX SYSTEMS OLED BOOT LOGO
   ======================= */
void ntxBootAnimation() {
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);

  int16_t x1, y1;
  uint16_t wNTX, hNTX;
  uint16_t wSYS, hSYS;

  display.setFont(&FreeSansBold9pt7b);
  display.getTextBounds("NTX", 0, 0, &x1, &y1, &wNTX, &hNTX);

  display.setFont(&FreeSans9pt7b);
  display.getTextBounds("SYSTEMS", 0, 0, &x1, &y1, &wSYS, &hSYS);

  int totalWidth = wNTX + 4 + wSYS;
  int startX = (SCREEN_WIDTH - totalWidth) / 2;
  int baselineY = 22;

  for (int x = -totalWidth; x <= startX; x += 2) {
    display.clearDisplay();

    display.setFont(&FreeSansBold9pt7b);
    display.setCursor(x, baselineY);
    display.print("NTX");

    display.setFont(&FreeSans9pt7b);
    display.setCursor(x + wNTX + 4, baselineY);
    display.print("SYSTEMS");

    display.display();
    delay(20);
  }

  for (int i = 0; i <= totalWidth; i += 4) {
    display.drawLine(startX, 30, startX + i, 30, SSD1306_WHITE);
    display.display();
    delay(8);
  }

  // LED boot wipe - a color band chases across the strip using the same
  // sine table as the main show, instead of a flat single-color flash
  for (int step = 0; step < NUMPIXELS + 6; step++) {
    for (int i = 0; i < NUMPIXELS; i++) {
      int d = step - i;
      if (d >= 0 && d < 6) {
        uint16_t hue = (uint16_t)(40000 + i * 2000);
        uint8_t val = (uint8_t)(255 - d * 40);
        pixels.setPixelColor(i, pixels.gamma32(pixels.ColorHSV(hue, 255, val)));
      } else {
        pixels.setPixelColor(i, 0);
      }
    }
    pixels.show();
    delay(25);
  }
  delay(150);
  pixels.clear();
  pixels.show();

  display.setFont(); // reset to default font for main show
}
`
### 4. Core concept: one shared clock drives everything 



