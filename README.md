# lt7680-drm for Raspberry Pi OS

DRM/KMS driver for the Levetop LT7680 on the EastRising ER-TFT3.95-1 (720x720), as a DKMS
module plus a device-tree overlay. **Not compiled and not tested on hardware.**

## Install

    sudo ./install.sh
    # then add to /boot/firmware/config.txt (or /boot/config.txt):
    dtoverlay=er-tft395
    sudo poweroff        # wire the display with the Pi off, then boot

`install.sh` installs `dkms`, `device-tree-compiler` and the kernel headers, builds and
installs the module, and compiles the overlay to `/boot/firmware/overlays/er-tft395.dtbo`.
It does not edit `config.txt`.

## Wiring (Pi 3/4/5, SPI0 CE0)

| Display pin | Signal | Pi GPIO | Header pin |
|---|---|---|---|
| 1, 2 | GND | GND | 6 |
| 3, 4 | VCC | 3.3V or 5V (per your module) | 1 or 2 |
| 5 | CS | GPIO8 (CE0) | 24 |
| 6 | MISO | GPIO9 | 21 |
| 7 | MOSI | GPIO10 | 19 |
| 8 | SCK | GPIO11 | 23 |
| 9 | BL | 3.3V (tutorial: HIGH = on) | 17 |
| 11 | RES | GPIO25 | 22 |
| 16 | panel CS | GPIO5 | 29 |
| 18 | panel SDI | GPIO13 | 33 |
| 19 | panel SCK | GPIO6 | 31 |

Other GPIOs: `dtoverlay=er-tft395,reset=24,pcs=5,psck=6,psdi=13,speed=16000000`.
If BL is wired to a GPIO instead of 3.3V, uncomment `backlight-gpios` in
`dts/er-tft395-overlay.dts` and reinstall (the driver then drives it high while the pipe is
enabled).

## Check

    dmesg | grep -i -E "lt7680|drm"
    ls /dev/dri /dev/fb*
    sudo apt install libdrm-tests fbset
    modetest -M lt7680                                  # connector and mode list
    cat /dev/urandom > /dev/fb1                         # noise (fb number may differ)
    sudo con2fbmap 1 1                                  # move the text console to this display

On Raspberry Pi OS Desktop the SPI panel shows up as a second DRM card next to the HDMI one;
it is not used by the desktop automatically.

## Before it works: checks

- Kernel config needs `DRM_KMS_HELPER` and `DRM_GEM_DMA_HELPER` (check with
  `zcat /proc/config.gz | grep -E "DRM_KMS_HELPER|DRM_GEM_DMA_HELPER"`; the stock kernel
  ships them for its tiny SPI display drivers). If `modprobe lt7680-drm` reports unknown
  symbols, one of them is missing.
- Written for 6.18.
- The pixel clock is rounded to whole MHz (36 MHz); `pixelclk-active = <0>` maps to the same
  register bit as the tutorial.
- Only the framebuffer path exists: no 2D engine, BTE, flash DMA or fonts.
- Module parameter `fifo_poll=0` skips the write-FIFO check between SPI chunks.
