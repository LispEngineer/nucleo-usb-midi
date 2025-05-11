# STM32F7 USB Midi

Attempt to make an STM32F7 USB device (Full Speed) as a MIDI interface.
This is *not* a host interface for USB MIDI.

Author:
* [Douglas P. Fields, Jr.](mailto:symbolics@lisp.engineer)
* Available under license: Apache 2.0
* New/modified code/docuemnts Copyright 2025 Douglas P. Fields, Jr.

Credits:
* [STM32 MIDI Box](https://github.com/Hypnotriod/midi-box-stm32)
  * License: BSD 2-Clause License
  * Copyright 2025 Illia Pikin a.k.a Hypnotriod
* ST Microelectronics
  * Nucleo F767ZI board
  * STM32 Cube IDE
  * STM32 libraries, etc.
* Segger
  * "ST-Link On Board J-Link" for Nucleo boards


# Initial IOC Configuration

The Nucleo board default IOC configuration include:

* USB_OTG_FS
  * Speed: Device, Full Speed
  * Low power: Disabled
  * Link power management: Disabled
  * VBUS sensing: Enabled
  * Signal start of frame: Enabled
  * Mode: Device_Only
  * USB On The Go FS global interrupt: *disabled*
  * PA8: USB_SOF [TP1]
  * PA9: USB_VBUS
  * PA11-12: USB Data ±
* Middleware
  * Default: NONE

# Initial Software Configuration and Test

First, we prove the USB port on the Nucleo board
works fine with known good code as provided by
default by the USB Middleware.


# Converting to USB MIDI Device

Now, we will follow the instructions in the
[STM32 MIDI Box](https://github.com/Hypnotriod/midi-box-stm32)
project to modify the USB device implementation to be
a MIDI device.