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
* [Cynthion](https://greatscottgadgets.com/cynthion/) USB Debugger


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

Make sure this builds and pushes using Segger J-Link.
(It does.)


# Initial Software Configuration and Test

First, we prove the USB port on the Nucleo board
works fine with known good code as provided by
default by the USB Middleware.

Update IOC for `USB_DEVICE` Middleware:
* FS IP: Human Interface Device Class (HID)
* Generate code
* Build, push to device

Confirm it shows up in [USBView](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/usbview#where-to-get-usbview):

```
[Port4]  :  USB Input Device


Is Port User Connectable:         yes
Is Port Debug Capable:            no
Companion Port Number:            4
Companion Hub Symbolic Link Name: USB#VID_14B0&PID_012D#5&3b72e4cf&0&17#{f18a0e88-c30c-11d0-8815-00a0c906bed8}
Protocols Supported:
 USB 1.1:                         yes
 USB 2.0:                         yes
 USB 3.0:                         no

Device Power State:               PowerDeviceD0

       ---===>Device Information<===---
English product name: "STM32 Human interface"

ConnectionStatus:                  
Current Config Value:              0x01  -> Device Bus Speed: Full (is not SuperSpeed or higher capable)
Device Address:                    0x1F
Open Pipes:                           1

          ===>Device Descriptor<===
bLength:                           0x12
bDescriptorType:                   0x01
bcdUSB:                          0x0201
bDeviceClass:                      0x00  -> This is an Interface Class Defined Device
bDeviceSubClass:                   0x00
bDeviceProtocol:                   0x00
bMaxPacketSize0:                   0x40 = (64) Bytes
idVendor:                        0x0483 = STMicroelectronics
idProduct:                       0x572B
bcdDevice:                       0x0200
iManufacturer:                     0x01
     English (United States)  "STMicroelectronics"
iProduct:                          0x02
     English (United States)  "STM32 Human interface"
iSerialNumber:                     0x03
     English (United States)  "3864355A3233"
bNumConfigurations:                0x01

          ---===>Open Pipes<===---

          ===>Endpoint Descriptor<===
bLength:                           0x07
bDescriptorType:                   0x05
bEndpointAddress:                  0x81  -> Direction: IN - EndpointID: 1
bmAttributes:                      0x03  -> Interrupt Transfer Type
wMaxPacketSize:                  0x0004 = 0x04 bytes
bInterval:                         0x0A

       ---===>Full Configuration Descriptor<===---

          ===>Configuration Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x02
wTotalLength:                    0x0022  -> Validated
bNumInterfaces:                    0x01
bConfigurationValue:               0x01
iConfiguration:                    0x00
bmAttributes:                      0xE0  -> Self Powered
  -> Remote Wakeup
MaxPower:                          0x32 = 100 mA

          ===>Interface Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x04
bInterfaceNumber:                  0x00
bAlternateSetting:                 0x00
bNumEndpoints:                     0x01
bInterfaceClass:                   0x03  -> HID Interface Class
bInterfaceSubClass:                0x01
bInterfaceProtocol:                0x02
iInterface:                        0x00

          ===>HID Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x21
bcdHID:                          0x0111
bCountryCode:                      0x00
bNumDescriptors:                   0x01
bDescriptorType:                   0x22 (Report Descriptor)
wDescriptorLength:               0x004A

          ===>Endpoint Descriptor<===
bLength:                           0x07
bDescriptorType:                   0x05
bEndpointAddress:                  0x81  -> Direction: IN - EndpointID: 1
bmAttributes:                      0x03  -> Interrupt Transfer Type
wMaxPacketSize:                  0x0004 = 0x04 bytes
bInterval:                         0x0A

          ===>BOS Descriptor<===
bLength:                           0x05
bDescriptorType:                   0x0F
wTotalLength:                      0x000C
bNumDeviceCaps:                    0x01

          ===>USB 2.0 Extension Descriptor<===
bLength:                           0x07
bDescriptorType:                   0x10
bDevCapabilityType:                0x02
bmAttributes:                      0x00000002  -> Supports Link Power Management protocol
```

Now, add code to prove that the device works:
Move the cursor +1 X pixel every quarter of
a second or so.

Build and push. It works.

Note that when you push and hold the `RESET` button on the Nucleo,
the board de-registers from USB. If you just tap it, it will
deregister and re-register quicly.


# Converting to USB MIDI Device

Now, we will follow the instructions in the
[STM32 MIDI Box](https://github.com/Hypnotriod/midi-box-stm32)
project to modify the USB device implementation to be
a MIDI device.

* Add the `usbd_midi.c` and `.h` files, including compiler include path settings
* Modify the `usb_device.c` per instructions
  * I used a `#define` and `#undef` since making the change requested
    would not persist after code generation (which I do a lot)
* Modify `main.h` per instructions

Major differences:

* The STM32F7 is very different than the STM32F1.
  * The STM32F1 uses a Packet Memory Area (PMA) for USB endpoint buffers. 
  * The STM32F7 utilizes a different mechanism for endpoint memory 
    management, and HAL_PCDEx_PMAConfig is not directly applicable. 

Now, the device shows up in USBView as:

```
[Port4]  :  USB Audio Device


Is Port User Connectable:         yes
Is Port Debug Capable:            no
Companion Port Number:            4
Companion Hub Symbolic Link Name: USB#VID_14B0&PID_012D#5&3b72e4cf&0&17#{f18a0e88-c30c-11d0-8815-00a0c906bed8}
Protocols Supported:
 USB 1.1:                         yes
 USB 2.0:                         yes
 USB 3.0:                         no

Device Power State:               PowerDeviceD0

       ---===>Device Information<===---
English product name: "MIDI LispEngineer"

ConnectionStatus:                  
Current Config Value:              0x01  -> Device Bus Speed: Full (is not SuperSpeed or higher capable)
Device Address:                    0x26
Open Pipes:                           2

          ===>Device Descriptor<===
bLength:                           0x12
bDescriptorType:                   0x01
bcdUSB:                          0x0201
bDeviceClass:                      0x00  -> This is an Interface Class Defined Device
bDeviceSubClass:                   0x00
bDeviceProtocol:                   0x00
bMaxPacketSize0:                   0x40 = (64) Bytes
idVendor:                        0x0483 = STMicroelectronics
idProduct:                       0x572B
bcdDevice:                       0x0200
iManufacturer:                     0x01
     English (United States)  "LispEngineer using STM"
iProduct:                          0x02
     English (United States)  "MIDI LispEngineer"
iSerialNumber:                     0x03
     English (United States)  "3864355A3233"
bNumConfigurations:                0x01

          ---===>Open Pipes<===---

          ===>Endpoint Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x05
bEndpointAddress:                  0x01  -> Direction: OUT - EndpointID: 1
bmAttributes:                      0x02  -> Bulk Transfer Type
wMaxPacketSize:                  0x0040 = 0x40 bytes
wInterval:                       0x0000
bSyncAddress:                      0x00

          ===>Endpoint Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x05
bEndpointAddress:                  0x81  -> Direction: IN - EndpointID: 1
bmAttributes:                      0x02  -> Bulk Transfer Type
wMaxPacketSize:                  0x0040 = 0x40 bytes
wInterval:                       0x0000
bSyncAddress:                      0x00

       ---===>Full Configuration Descriptor<===---

          ===>Configuration Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x02
wTotalLength:                    0x0053  -> Validated
bNumInterfaces:                    0x01
bConfigurationValue:               0x01
iConfiguration:                    0x00
bmAttributes:                      0x80  -> Bus Powered
MaxPower:                          0xFA = 500 mA

          ===>Interface Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x04
bInterfaceNumber:                  0x00
bAlternateSetting:                 0x00
bNumEndpoints:                     0x02
bInterfaceClass:                   0x01  -> Audio Interface Class
bInterfaceSubClass:                0x03  -> MIDI Streaming Interface SubClass
bInterfaceProtocol:                0x00
iInterface:                        0x00

          ===>Descriptor Hex Dump<===
bLength:                           0x07
bDescriptorType:                   0x24
07 24 01 00 01 41 00 

          ===>Descriptor Hex Dump<===
bLength:                           0x06
bDescriptorType:                   0x24
06 24 02 02 01 00 

          ===>Descriptor Hex Dump<===
bLength:                           0x09
bDescriptorType:                   0x24
09 24 03 01 02 01 01 01 00 

          ===>Descriptor Hex Dump<===
bLength:                           0x06
bDescriptorType:                   0x24
06 24 02 01 03 00 

          ===>Descriptor Hex Dump<===
bLength:                           0x09
bDescriptorType:                   0x24
09 24 03 02 04 01 03 01 00 

          ===>Endpoint Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x05
bEndpointAddress:                  0x01  -> Direction: OUT - EndpointID: 1
bmAttributes:                      0x02  -> Bulk Transfer Type
wMaxPacketSize:                  0x0040 = 0x40 bytes
wInterval:                       0x0000
bSyncAddress:                      0x00

          ===>Descriptor Hex Dump<===
bLength:                           0x05
bDescriptorType:                   0x25
05 25 01 01 03 

          ===>Endpoint Descriptor<===
bLength:                           0x09
bDescriptorType:                   0x05
bEndpointAddress:                  0x81  -> Direction: IN - EndpointID: 1
bmAttributes:                      0x02  -> Bulk Transfer Type
wMaxPacketSize:                  0x0040 = 0x40 bytes
wInterval:                       0x0000
bSyncAddress:                      0x00

          ===>Descriptor Hex Dump<===
bLength:                           0x05
bDescriptorType:                   0x25
05 25 01 01 02 

          ===>BOS Descriptor<===
bLength:                           0x05
bDescriptorType:                   0x0F
wTotalLength:                      0x000C
bNumDeviceCaps:                    0x01

          ===>USB 2.0 Extension Descriptor<===
bLength:                           0x07
bDescriptorType:                   0x10
bDevCapabilityType:                0x02
bmAttributes:                      0x00000002  -> Supports Link Power Management protocol
```

Now, send MIDI notes using the library's function. This works.
You can see them in [MidiView](https://hautetechnique.com/midi/midiview/).

## Sending to a second USB MIDI interface

* Send MIDI notes on USB MIDI device interface 2
  * This is as simple as setting the "Cable" setting
    of the 32-bit USB MIDI 1.0 packet.
  * Multiple "cables" do not use multiple USB Endpoints.

## Changing the name of the second USB MIDI Interface

Turn off USB HID:
* `Middlewares/ST/STM32_USB_Device_Library/Class/HID`
  * Right click, pick properties
  * `C/C++ Build` -> `[All Configurations]`
  * Exclude resource from build: CHECKED
  * Clean & build project
* Now we can simply delete the `HID` directory
  * Remove references to `usb_hid.h` in various includes
  * Clean & build

Files:
* `usbd_desc.c` contains various strings in `USBD_DESC_Private_Defines` section
  * Example: `USBD_PRODUCT_STRING_FS` (value `MIDI LispEngineer`)
    * Read by `USBD_FS_ProductStrDescriptor`
    * Included in `USBD_DescriptorsTypeDef FS_Desc` in `usbd_desc.c`
* This structure `FS_Desc` is set in `USBD_Init()` of `usbd_core.c`
  * It becomes part of `USBD_HandleTypeDef -> pDesc`
* It's then referenced in `usbd_ctlreq.c` in `USBD_GetDescriptor()`
  * `USB_DESC_TYPE_STRING` for the `req->wValue >> 8` (a 16-bit unsigned int)
  * Then the bottom byte is the string descriptor ID, example `USBD_IDX_LANGID_STR`

Descriptors in `FSC_Desc`:
* 0: Device
* 1: LangID Str
* 2: Manufacturer Str
* 3: Product STr
* 4: Serial Str
* 5: Config Str
* 6: Interface Str
* 7: (LPM) USR_BOS

So, let me do this:
* Create new `USBD_IDX_MIDI_OUT2_STR` with `0x10U` in `usbd_def.h`
* Add stuff in `usbd_desc.c`
  * New String Define `USBD_MIDI_OUT2_FS`
  * New function `USBD_FS_MIDIStrDescriptor()` that uses that string
  * Add the definition in `usbd_desc.h`
    * Include it in `usbd_ctlreq.c`
* Edit `USBD_GetDescriptor` in the section of `USB_DESC_TYPE_STRING` in `usbd_ctlreq.c`
  * Add new `case` for `USBD_IDX_MIDI_OUT2_STR`
  * Call that new function above
* At this point, we have now defined a new string descriptor #0x10, I believe -
  code should build and run fine, but nothing different happens.
* Add the `USBD_IDX_MIDI_OUT2_STR` to the appropriate `iJack` fields of the
  MIDI IN Jack Descriptor
  * As far as I can tell, this does nothing in Windows.
  * TODO: Test MacOS, Linux, etc. (Not that I have MacOS devices.)

Windows:
* Per Gemini: **Windows**: Historically, Windows has been inconsistent in displaying 
  these individual `iJackName` strings directly as the port names in 
  Digital Audio Workstations (DAWs) or MIDI utilities. It often defaults to a pattern like 
  "[Product Name] Port 1", "[Product Name] Port 2", etc., or 
  "[Interface Name] Port 1". However, the custom names provided via `iJackName` might be 
  visible in the Windows Device Manager (often under "Sound, video and game controllers" 
  or sometimes "Software Devices" if a specific driver model is used). Recent updates 
  to Windows MIDI Services are aiming for better naming fidelity.
* See also [Windows MIDI Services October 2024 Update](https://devblogs.microsoft.com/windows-music-dev/windows-midi-services-oct-2024-update/)
  or [Windows MIDI on GitHub](https://microsoft.github.io/MIDI/)
  * This stuff won't be properly available until end of 2025 it seems
* It does seem that Windows queries the two strings due to the red LED
  flashing upon board reset. I should verify this with my Cynthion.

Mac:
* I can see the Name showing up on an M3 MacBook Air running macOS 14.6.
  * It is showing up as unicode junk though
  * It only shows up in [WebMIDI Tester](https://studiocode.dev/webmidi-tester/),
    not in `Audio MIDI Setup`
    * `input MIDI LispEngineer Port 1`
    * `input MIDI LispEngineer <garbled>`
    * `output MIDI LispEngineer Port 1`
  * This is due to a bug in `USBD_GetDescriptor()` which I fixed, and now it works!
* Note: Mac will not update the name of a MIDI Jack unless you create a new configuration
  in the MIDI Studio from scratch, so beware of that if you try to change your USB
  device on the fly.

TODO:
* Figure out if it is the External or Embedded MIDI IN/OUT Jack Descriptor
  that gets the name that shows up in Apple MIDI
  * The possibilities are labeled as `It's either this one` or
    `Or this one` in the code

# Open Questions

* How do we know when we can submit a new IN packet for USB transfer?
  * How do we know when the current IN packet is acknowledged & delivered?
  * How do we know when the current IN packet is requested?
  * Is there an atomic mechanism to replace the current IN packet with a
    different one, prior to it being picked up?
  * Is there a way to cancel a previously scheduled IN packet before it
    is sent (or started to be sent)?