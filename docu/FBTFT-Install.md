# FBTFT Framebuffer Installation

## Automatic Installation for RPi-Display (with 8-Bit SPI)
Download the installation script ```rpi-display.sh``` and run it as root. The last parameter sets the rotation and can be 0, 90, 180 or 270.
```
$ wget -N https://github.com/watterott/RPi-Display/raw/master/rpi-display.sh
$ sudo /bin/bash rpi-display.sh 270
```
To start the desktop run:
```
$ startx -- -layout TFT
$ startx -- -layout HDMI
```
*If ```-layout``` is not set, then the TFT display will be used as default.*

## Manual Installation

### 1. [Install FBTFT](https://github.com/notro/fbtft/wiki#install) Framebuffer and Touchscreen Tools

* #### Raspbian

    The latest official Raspberry Pi Device Tree enabled Kernel includes FBTFT, so you need an up to date system.
    To update your system run:
    ```
    $ sudo apt-get update
    $ sudo REPO_URI=https://github.com/notro/rpi-firmware rpi-update
    $ sudo reboot
    ```

* #### Other Linux Systems

    Update the Kernel with:
    ```
    $ sudo curl -L --output /usr/bin/rpi-update https://raw.githubusercontent.com/Hexxeh/rpi-update/master/rpi-update && sudo chmod +x /usr/bin/rpi-update
    $ sudo REPO_URI=https://github.com/notro/rpi-firmware rpi-update
    $ sudo reboot
    ```
    *On compatibility errors run rpi-update as follows:
    ```sudo -E RPI_UPDATE_UNSUPPORTED=0 REPO_URI=https://github.com/notro/rpi-firmware rpi-update```*

* #### Touchscreen Tools

    To install the touchscreen tools run:
    ```
    $ sudo apt-get install tslib libts-bin
    $ sudo apt-get install xinput
    $ cd /tmp
    $ wget http://tronnes.org/downloads/xinput-calibrator_0.7.5-1_armhf.deb
    $ sudo dpkg -i -B xinput-calibrator_0.7.5-1_armhf.deb
    $ rm xinput-calibrator_0.7.5-1_armhf.deb
    ```

### 2. Activate FBTFT

*Note: For a higher speed than 16MHz the display has to be connected directly to the Raspberry Pi or with wires not longer than 5cm.*

Choose your respective Kernel version.
The default on the latest Raspbian is a Device Tree enabled Kernel.

* #### FBTFT Device Tree enabled Kernel

    ##### RPi-Display (8-Bit SPI)
    Open the file ```/boot/config.txt```:
    ```
    $ sudo nano /boot/config.txt
    ```
    Add the following line at the file end (rotate can be 0, 90, 180, 270):
    ```
    dtoverlay=rpi-display,speed=32000000,rotate=270
    ```
    *The default parameters are ```speed=32000000, rotate=270```.*

    Reboot the system:
    ```
    $ sudo reboot
    ```

* #### FBTFT Kernel Modules

    To use the fbtft_device module instead of a DT overlay on a Device Tree enabled Kernel, add the following line to ```/boot/config.txt```:
    ```
    dtparam=spi=on
    ```

    If the SPI module (**spi-bcm2708**) is not loaded, remove or comment out the blacklist line in ```/etc/modprobe.d/raspi-blacklist.conf```.
    To load the module by hand run ```sudo modprobe spi-bcm2708```.

    ##### RPi-Display (8-Bit SPI)
    ```
    $ sudo modprobe fbtft_device name=rpi-display speed=32000000 rotate=270
    ```
    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    fbtft_device name=rpi-display speed=32000000 rotate=270
    ```

    ##### RPi-Display (9-Bit SPI)
    *Note: Only first generation RPi-Displays before April 2014 use 9-Bit SPI.*
    ```
    $ sudo modprobe fbtft_device name=mi0283qt-9a gpios=reset:23,led:18 speed=32000000 rotate=270
    ```
    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    fbtft_device name=mi0283qt-9a gpios=reset:23,led:18 speed=32000000 rotate=270
    ```

    ##### MI0283QT-Adapter v1 on RPi-ShieldBridge v1 (9-Bit SPI)
    ```
    $ sudo modprobe fbtft_device name=mi0283qt-9a gpios=reset:23,led:24 speed=32000000 rotate=270
    ```
    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    fbtft_device name=mi0283qt-9a gpios=reset:23,led:24 speed=32000000 rotate=270
    ```

    ##### MI0283QT-Adapter v2 on RPi-ShieldBridge v1 (8-Bit SPI)
    ```
    $ sudo modprobe fbtft_device name=mi0283qt-v2 gpios=reset:23 speed=16000000 rotate=270
    ```
    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    fbtft_device name=mi0283qt-v2 gpios=reset:23 speed=16000000 rotate=270
    ```

    ##### ADS7846 Touchcontroller
    ```
    $ sudo modprobe ads7846_device model=7846 cs=1 gpio_pendown=25 speed=2000000 keep_vref_on=1 x_plate_ohms=60 pressure_max=255 x_min=200 x_max=3900 y_min=200 y_max=3900
    ```
    To make it permanent (on Debian) add to the file ```/etc/modules``` the following line:
    ```
    ads7846_device model=7846 cs=1 gpio_pendown=25 speed=2000000 keep_vref_on=1 x_plate_ohms=60 pressure_max=255 x_min=200 x_max=3900 y_min=200 y_max=3900
    ```
    To make the touchcontroller accessible as char device ```/dev/input/touchscreen``` create a new file  ```/etc/udev/rules.d/95-ads7846.rules``` and add the following line:
    ```
    SUBSYSTEM=="input", KERNEL=="event[0-9]*", ATTRS{name}=="ADS7846 Touchscreen", SYMLINK+="input/touchscreen"
    ```

* #### FBTFT compiled into Kernel (BRANCH=builtin)

    ##### RPi-Display (8-Bit SPI)
    Add the following Kernel arguments to ```/boot/cmdline.txt```:
    ```
    fbtft_device.name=rpi-display fbtft_device.speed=32000000 fbtft_device.rotate=270
    ```

    ##### RPi-Display (9-Bit SPI)
    *Note: Only first generation RPi-Displays before April 2014 use 9-Bit SPI.*
    Add the following Kernel arguments to ```/boot/cmdline.txt```:
    ```
    fbtft_device.name=mi0283qt-9a fbtft_device.speed=32000000 fbtft_device.gpios=reset:23,led:18 fbtft_device.rotate=270
    ```

    ##### MI0283QT-Adapter v1 on RPi-ShieldBridge v1 (9-Bit SPI)
    Add the following Kernel arguments to ```/boot/cmdline.txt```:
    ```
    fbtft_device.name=mi0283qt-9a fbtft_device.speed=32000000 fbtft_device.gpios=reset:23,led:24 fbtft_device.rotate=270
    ```

    ##### MI0283QT-Adapter v2 on RPi-ShieldBridge v1 (8-Bit SPI)
    Add the following Kernel arguments to ```/boot/cmdline.txt```:
    ```
    fbtft_device.name=mi0283qt-v2 fbtft_device.speed=16000000 fbtft_device.gpios=reset:23 fbtft_device.rotate=270
    ```

### 3. Enable for Console

Run in console (not desktop terminal):
```
$ con2fbmap 1 1
```
To make it permanent (on Debian) add to the file ```/boot/cmdline.txt``` at the end of the line the following Kernel arguments: ```fbcon=map:10 fbcon=font:VGA8x8```

### 4. Enable for X-Window-System
If you have not used the automatic installation script then
open ```99-fbturbo.conf``` and change *fb0* to *fb1* (only needed once):
```
$ sudo nano /usr/share/X11/xorg.conf.d/99-fbturbo.conf
```

Start X-Window-System:
```
$ startx &
```

...wait till X-Window-System starts up and set the ADS7846 properties:
* rotate=0 - no settings needed

* rotate=90
    ```
    $ DISPLAY=:0 xinput --set-prop 'ADS7846 Touchscreen' 'Evdev Axes Swap' 1
    $ DISPLAY=:0 xinput --set-prop 'ADS7846 Touchscreen' 'Evdev Axis Inversion' 1 0
    ```

* rotate=180
    ```
    $ DISPLAY=:0 xinput --set-prop 'ADS7846 Touchscreen' 'Evdev Axis Inversion' 1 1
    ```

* rotate=270
    ```
    $ DISPLAY=:0 xinput --set-prop 'ADS7846 Touchscreen' 'Evdev Axes Swap' 1
    $ DISPLAY=:0 xinput --set-prop 'ADS7846 Touchscreen' 'Evdev Axis Inversion' 0 1
    ```

...to stop X-Window-System:
```
$ sudo pkill x
```

To make it permanent (on Debian) add to the file ```/etc/X11/xinit/xinitrc``` the respective settings.
Further infos: [xinput settings](https://github.com/notro/fbtft/wiki/Touchpanel#-xinput---make-it-permanent) and [autostart x](https://github.com/notro/fbtft/wiki#make-it-permanent-debian).

### 5. Video Test

*Note: The video file is about 60MB big.*
```
$ wget http://download.blender.org/peach/bigbuckbunny_movies/BigBuckBunny_320x180.mp4
```

Play with **mplayer**:
```
$ sudo apt-get install mplayer
$ mplayer -vo fbdev2:/dev/fb1 -vf scale=320:-3 BigBuckBunny_320x180.mp4
```

Play with **omxplayer** (*fbcp* for framebuffer mirroring required):
```
$ fbcp &
$ omxplayer BigBuckBunny_320x180.mp4
$ killall fbcp
```

### 6. Calibrate Touchpanel (optional)

For better accuracy a touchpanel calibration can be done with:

* ### xinput_calibrator

    ```
    $ sudo DISPLAY=:0 xinput_calibrator --device "ADS7846 Touchscreen" --output-type xinput
    ```
    For axes swapping and inversion see [Enable for X-Window-System](https://github.com/watterott/RPi-Display/blob/master/docu/FBTFT-Install.md#4-enable-for-x-window-system).

* ### ts_calibrate

    ```
    $ sudo TSLIB_FBDEVICE=/dev/fb1 TSLIB_TSDEVICE=/dev/input/touchscreen ts_calibrate
    $ sudo TSLIB_FBDEVICE=/dev/fb1 TSLIB_TSDEVICE=/dev/input/touchscreen ts_test
    ```

### 7. Further Infos
* [FBTFT on Raspian](https://github.com/notro/fbtft/wiki/FBTFT-on-Raspian)
* [FBTFT Setup on modern Raspbian](http://lallafa.de/blog/2015/03/fbtft-setup-on-modern-raspbian/)
* [FBTFT on Arch Linux](http://philipp-weissmann.de/touch-display-auf-raspberry-pi-mit-arch-linux.html)
