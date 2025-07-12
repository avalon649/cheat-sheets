### Flashing via USB 

If your SLZB-06/06M device is not connected to LAN or you want to flash ESP32 via USB for other reasons, it can be easily done via USB cable. For this purposes you need:

A computer with Windows or Mac installed;
USB - Type-C cable.

To flash via USB, follow these steps:

1. Install esptool

```bash
pip install esptool
```

2. Check if the ESP is connected

```bash
esptool.py --port /dev/ttyUSB0 chip_id
```
3. Download the flashing software. We recommend using ESP_Flasher, which can be downloaded from [this fork of official repository at Github](https://github.com/smlight-dev/ESP_Flasher)

4. Download the latest firmware version from [official SLZB-06 Firmware flasher](https://smlight.tech/flasher)

5. Be sure, that you have downloaded and installed the drivers for USB/UART converter, built in to your SLZB-06/06M.

* for CP2102 USB/UART converter please use the latest version from the [official web-page here](https://www.silabs.com/interface/usb-bridges/classic/device.cp2102?tab=softwareandtools)

* for СH9102 USB/UART converter please use the latest version from the [official web-page here](http://www.wch-ic.com/search?q=CH9102&t=downloads)

6. Run the ESP_Flasher program and

List all available ports

```bash
ls /dev/ttyUSB0
```
Run this command To flash the firmware 

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x0 firmware.bin
```

7. Wait for the firmware to complete.

