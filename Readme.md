# HardDucky

**HardDucky** is a security tool designed to detect and block malicious or "bad" USB devices (such as malicious HID devices) on Linux systems. It monitors input devices for suspicious keyboard activity (e.g., rapid key presses typical of automated HID attacks) and automatically disables the offending USB device by unbinding its USB driver and adding udev rules to prevent reconnection.

---

## Features

* Monitors all input devices for suspicious keyboard activity (excessive key presses per second).
* Automatically blocks detected malicious USB devices by unbinding their USB driver.
* Maintains a blacklist of blocked USB device IDs.
* Creates persistent udev rules to prevent blocked devices from reconnecting.
* Includes command-line tools to list USB devices, list blocked devices, and manually ban or unban devices by Vendor ID and Product ID.
* Configurable parameters via `/etc/hardducky/hardducky.conf`.
* Logs detected violations for audit purposes.

---

## Installation

The project files are organized as a Debian package structure:

```
hardducky/
├── DEBIAN/
│   ├── control
│   └── postinst
├── etc/hardducky/hardducky.conf
├── lib/systemd/system/hardducky.service
└── usr/bin/
    ├── hardducky
    └── hardducky-tools
```

* Install the package using `dpkg -i hardducky.deb`.
* Enable and start the `hardducky.service` for automatic monitoring on boot.

---

## Configuration

The main configuration file is located at:

```
/etc/hardducky/hardducky.conf
```

Typical configurable options:

* `max_keys_per_second`: Maximum allowed key presses per second before considering an attack (default: 2).
* `log_file`: Path to log detected attacks (default: `/var/log/hardducky.log`).

---

## Usage

### Running the Monitor

The main daemon script (`hardducky`) runs continuously to monitor USB input devices:

```bash
sudo /usr/bin/hardducky
```

It requires root privileges to unbind USB drivers and manage udev rules.

### Command-Line Tools (`hardducky-tools`)

The `hardducky-tools` script allows manual management of blocked USB devices:

```bash
sudo /usr/bin/hardducky-tools <command> [vendor_id] [product_id]
```

Available commands:

* `list-usb`: Lists currently connected USB devices (similar to `lsusb`).
* `list-blocked`: Lists USB devices currently blocked by HardDucky.
* `ban <vendor_id> <product_id>`: Manually block a USB device by its Vendor ID and Product ID.
* `unban <vendor_id> <product_id>`: Remove a device from the blocklist and re-enable it.

Example:

```bash
sudo hardducky-tools ban 046d c534
sudo hardducky-tools unban 046d c534
```

---

## How It Works

* The daemon monitors keyboard input devices using the `evdev` interface.
* If a device sends more key presses than the configured threshold within one second, it is identified as suspicious.
* The USB device ID is determined via sysfs paths.
* The device is unbound from its USB driver to disable it.
* The USB device ID is added to a blacklist file (`/etc/usb_blocklist.conf`).
* A persistent udev rule is created to block the device from reconnecting after reboot.

---

## Requirements

* Linux system with Python 3.
* `evdev` Python library installed.
* Root privileges to run the monitor and manage USB device drivers.
* Systemd for running the `hardducky.service`.

---

## Logs

Detected attacks are logged to the configured log file (default `/var/log/hardducky.log`), recording the time, device name, offending key, and the rate of key presses.

---

## Security Considerations

* Designed to protect against malicious USB devices, especially those impersonating keyboards (like USB Rubber Ducky devices).
* Requires root to enforce device blocking at the system level.
* Blocking is persistent across reboots through udev rules.
