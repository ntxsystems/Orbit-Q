# Fixing ST-LINK USB Permissions on Ubuntu Linux Using udev Rules

## STM32CubeProgrammer Troubleshooting Guide

**Platform:** Ubuntu Linux  
**Tool:** STM32CubeProgrammer  
**Interface:** ST-LINK  
**Topic:** Installing and reloading udev rules to resolve USB permission issues

---

## 1. Introduction

ST-LINK is a debugging and programming interface used to communicate with STM32 microcontrollers. On Linux, incorrect or missing USB device permissions can prevent STM32CubeProgrammer from detecting or connecting to an ST-LINK device.

This tutorial explains how to locate the ST-LINK udev rules, copy them into the system rules directory, reload the rules, and verify USB device detection.

## 2. Prerequisites

Before starting, ensure that you have:

- Ubuntu Linux installed.
- STM32CubeProgrammer installed.
- An ST-LINK debugger or compatible development board.
- A USB cable capable of data transfer.
- Administrator privileges for running commands with `sudo`.

## 3. Procedure

### Step 1: Check the existing udev rules directory

Open Terminal using `Ctrl + Alt + T` and execute:

```bash
ls -l /etc/udev/rules.d/
```

**Purpose:** Lists the existing udev rules installed on the system.

If the ST-LINK rules are not present, continue to the next step.

### Step 2: Search for the ST-LINK rules file

Search the home directory for files containing ST-LINK rules:

```bash
find ~ -type f -iname "*.rules" 2>/dev/null | grep -i stlink
```

If no results appear, search the entire system:

```bash
sudo find / -type f -iname "*.rules" 2>/dev/null | grep -i stlink
```

**Purpose:** Locates the rules file that may be included with the STM32CubeProgrammer installation.

If a file path appears, note the complete path for use in Step 4.

### Step 3: Locate the STM32CubeProgrammer installation

If the rules file cannot be found, search for the installation directory:

```bash
find /opt /usr/local /home -type d -iname "*cubeprogrammer*" 2>/dev/null
```

Once the installation directory is identified, navigate to it using the appropriate path. For example:

```bash
cd "/home/yourusername/STMicroelectronics/STM32Cube/STM32CubeProgrammer"
```

**Note:** Replace the example path with the actual installation path on your system.

Locate the rules directory:

```bash
find . -type d -path "*/Driver/rules"
```

This command searches for the `Driver/rules` directory inside the current directory.

### Step 4: Copy the rules into the system directory

If the `Driver/rules` directory is available, copy its contents into `/etc/udev/rules.d/`:

```bash
sudo cp /path/to/Driver/rules/* /etc/udev/rules.d/
```

**Important:** Replace `/path/to/Driver/rules` with the actual path returned in Step 3.

If you located a single rules file directly, you can copy it using:

```bash
sudo cp "/YOUR/ACTUAL/PATH/file.rules" /etc/udev/rules.d/
```

Replace `/YOUR/ACTUAL/PATH/file.rules` with the exact file path found on your system.

**Purpose:** Places the ST-LINK rules in the directory where Linux reads system-wide udev rules.

### Step 5: Reload the udev rules

After copying the rules, execute:

```bash
sudo udevadm control --reload-rules
```

Then execute:

```bash
sudo udevadm trigger
```

**Purpose:**

- `udevadm control --reload-rules` reloads the udev rules database.
- `udevadm trigger` requests processing of device events using the available rules.

### Step 6: Install the USB development library

Install the USB development package if it is not already installed:

```bash
sudo apt update
sudo apt install libusb-1.0-0-dev
```

**Purpose:** Installs the USB development library package used by USB-related software.

### Step 7: Reconnect the ST-LINK device

1. Disconnect the ST-LINK USB cable.
2. Wait approximately five seconds.
3. Reconnect the USB cable.
4. Ensure the development board is powered and connected properly.

Check whether Linux detects the USB device:

```bash
lsusb
```

Look for a device identified as STMicroelectronics or ST-LINK in the output.

Check the installed rules:

```bash
ls -l /etc/udev/rules.d/*stlink*
```

**Note:** If this last command reports that no matching file exists, the installed rules may use a different filename. Check the files copied in Step 4.

### Step 8: Test STM32CubeProgrammer

1. Close STM32CubeProgrammer completely.
2. Reopen the application.
3. Select **ST-LINK** as the connection interface.
4. Try connecting to the target microcontroller.

If the connection succeeds, the issue may have been related to the missing or incorrectly installed udev rules.

If the connection still fails, investigate USB detection, device permissions, the cable, board power, and the ST-LINK firmware or hardware.

---

## 4. Troubleshooting Summary

| Issue | Recommended action |
|---|---|
| No rules file found | Search for the installation directory and `Driver/rules`. |
| Rules copied but connection still fails | Reload udev rules and reconnect the device. |
| ST-LINK not visible in `lsusb` | Check the USB cable, port, board power, and hardware. |
| Rules file not listed by the final command | Verify the installed filename and copied files. |
| STM32CubeProgrammer still cannot connect | Check permissions and investigate the connection error. |

## 5. Conclusion

Installing the appropriate udev rules allows Linux to apply the required device permissions and handling rules for ST-LINK. Reloading the rules and reconnecting the device helps ensure that the updated configuration is applied.

Successful USB detection is an important diagnostic step, but it does not by itself guarantee a successful ST-LINK connection.

## 6. Official Reference

- STMicroelectronics, [STM32CubeProgrammer Installation Documentation](https://dev.st.com/stm32cube-docs/prog/2.23.0/en/docs/markup/CubeProg_How_To_Start/CubeProg_Installation.html)
- STMicroelectronics, [STM32CubeProgrammer User Manual](https://www.st.com/resource/en/user_manual/um2237-stm32cubeprogrammer-software-description-stmicroelectronics.pdf)

---

**Documentation purpose:** Linux troubleshooting for ST-LINK USB permissions and udev rules during STM32CubeProgrammer setup.
