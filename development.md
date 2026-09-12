## ESP32-C3 USB CDC

For serial output via the integrated USB-JTAG/CDC port, the following build flags are required (unlike the ESP32-S3):

```ini
build_flags =
    -D ARDUINO_USB_MODE=1
    -D ARDUINO_USB_CDC_ON_BOOT=1
```

## Solve ACM0/1 switching after upload

To avoid relying on changing `/dev/ttyACM*` device names after upload/reset, use the stable Linux serial-by-id path and keep DTR/RTS low during the monitor/reset procedure.

Example PlatformIO configuration:

```ini
[env:esp32c3]
platform = espressif32
board = esp32-c3-devkitm-1
framework = arduino

upload_port = /dev/serial/by-id/usb-Espressif_USB_JTAG_serial_debug_unit_<DEVICE-ID>-if00
monitor_port = /dev/serial/by-id/usb-Espressif_USB_JTAG_serial_debug_unit_<DEVICE-ID>-if00

monitor_speed = 115200
upload_speed = 460800

monitor_dtr = 0
monitor_rts = 0

build_flags =
    -D ARDUINO_USB_MODE=1
    -D ARDUINO_USB_CDC_ON_BOOT=1
```

Use `ls -l /dev/serial/by-id/` to determine the actual persistent device path on the development system.
