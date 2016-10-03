esp-open-rtos Build Environment
===============================

[![Docker Pulls](https://img.shields.io/docker/pulls/bschwind/esp-open-rtos.svg)](https://hub.docker.com/r/bschwind/esp-open-rtos/) [![Docker Stars](https://img.shields.io/docker/stars/bschwind/esp-open-rtos.svg)](https://hub.docker.com/r/bschwind/esp-open-rtos/) [![](https://images.microbadger.com/badges/image/bschwind/esp-open-rtos.svg)](https://microbadger.com/images/bschwind/esp-open-rtos "Get your own image badge on microbadger.com") [![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/bschwind/esp-build/blob/master/LICENSE)

This Dockerfile contains the dependencies necessary to create a toolchain for the ESP8266 chip.
It is based on [esp-open-rtos](https://github.com/SuperHouse/esp-open-rtos) and allows for easy building and flashing
to the ESP8266 chip for projects written with esp-open-rtos.

Dependencies
------------
- [Docker](https://www.docker.com/products/docker-toolbox)
- [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (if you're not on Linux)

Quick Setup
-----------

* `docker pull bschwind/esp-open-rtos`
* `cd` to your esp-open-rtos project
* Without USB flashing support: `docker run --rm -it -e "ESPBAUD=921600" -v $(PWD):/home/esp/esp-open-rtos/examples/project bschwind/esp-open-rtos /bin/bash`
* With USB flashing support: `docker run --rm -it --privileged -e "ESPBAUD=921600" -v /dev/bus/usb:/dev/bus/usb -v $(PWD):/home/esp/esp-open-rtos/examples/project bschwind/esp-open-rtos /bin/bash`

Either step will put you in an interactive shell inside the container. If you have a Makefile in your project directory, you can immediately
run `make` and your source should get compiled. `make flash` will attempt to flash the code to `/dev/ttyUSB0`, assuming you're using the
Makefiles from esp-open-rtos examples.

Flashing Images from the Container
----------------------------------

If you're on docker-machine (OS X or Windows), you need to forward your USB device within Virtualbox. This is best managed in the VirtualBox GUI.

Steps:

* Stop your docker virtual machine host, if applicable
* Plug in the USB serial device you will use to flash to the ESP8266
* Install [virtualbox extensions](https://www.virtualbox.org/wiki/Downloads) to support USB (Ctrl-F "extension" on that page)
  * OS X -> Under "Virtualbox" -> Preferences, go to the Extensions tab
  * Windows -> Same thing?
  * Click the "Adds new package" button and select the extension pack you downloaded
* Return to the main VirtualBox GUI
* Right click on your docker VM and select "Settings"
* Select "Ports" -> "USB"
* Check the box "Enable USB Controller" and select "USB 2.0 (EHCI) Controller"
* Under "USB Device Filters" click the USB icon with the green plus sign to add a USB device
* Select your USB serial device (in my case it was "FTDI FT232R USB UART [0600]")
* Click OK until you're back to the main Virtualbox GUI
* At this point you can restart your virtual machine with `docker-machine start <YOUR_DOCKER_VM_NAME>`
* Run docker as we did in Quick Setup: `docker run --rm -it --privileged -v /dev/bus/usb:/dev/bus/usb -v $(PWD):/home/esp/esp-open-rtos/examples/project bschwind/esp-open-rtos /bin/bash`
  * NOTE: With the `-v /dev/bus/usb:/dev/bus/usb` volume, the `/dev/bus/usb` on the lefthand side of the colon refers to docker VM's USB directory, *not* your host machine (you likely won't find that path on OS X)
* `/dev/ttyUSB0` should now be available
* Run `make` and then `make flash` on an example project or your own

If you're on Linux, it should be sufficient to share your USB device either as a docker volume or with the `--device` flag. However, I have not yet tested Linux.

Serial Debugging
----------------

[Picocom](https://github.com/npat-efault/picocom) is installed in this image by default. Invoke it with `picocom -b 115200 /dev/ttyUSB0` (change the baud rate and device path accordingly)

Stop it with `Ctrl-A Ctrl-X`

PRO TIP
-------

When running `make flash` from esp-open-rtos's Makefiles, it will print out the flashing command it uses, for example `esptool.py -p /dev/ttyUSB0 --baud 115200 write_flash -fs 16m -fm qio -ff 40m 0x0 ../../bootloader/firmware_prebuilt/rboot.bin 0x1000 ../../bootloader/firmware_prebuilt/blank_config.bin 0x2000 ./firmware/blink_timers.bin`

See that `--baud 115200`? You can change it to higher values with the ESPBAUD environment variable which is passed into the `docker run` command and the ESP8266 should honor it (in most cases?). Valid values I've had success with are [230400, 460800, 921600]
with 921600 flashing in under 4 seconds (241,664 bytes). This can significantly speed up development time.

With any flashing baud rate, I have noticed more occasional errors with this setup than I have with the Arduino IDE or other environments, I'm not sure what the cause is.
The flash will sometimes get stuck at 99% with `A fatal error occurred: Timed out waiting for packet header`. I've found this actually happens *less* often on 921600 baud,
but it can still happen. If anyone knows what's up with that, please let me know!
 