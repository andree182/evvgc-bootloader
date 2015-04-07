# Introduction

This is a small fork of the original bootloader from maple, which is further based
on the stm32 bootloader ideas... Quite a journey.

The original evvgc usb bootloader didn't seem to work with evvgc-plus, and the
code was closed - hence this second spin-off.

# Usage

1. make
2. flash bootloader to evvgc via uart1 (*LiPo power off!*):

    make program_serial
3. build target system to be linked from 0x8004000. For evvgc-plus, this is done by:

    make USE_USB_BL=yes
4. (remove BOOT1 pin and) reboot board
5. flash the target system using dfu_util (remember - you only have ~2s after powerup to flash, if there is some valid firmware flashed already):

    dfu-util --alt 1  --download $(IMAGE_PATH).bin -R
6. blue led (on newer boards) blinks once at boot, then multiple times if image is booted. No blue blinks == bad.

# TODO
* allow reboot to bootloader (some USB magic needs to be added for the host to correctly reset and detect change)
* allow forcing staying in bootloader mode even if there is a valid firwmare (for convenient re-flashing)

# Maple bootloader info
## Code structure

stm32lib/*
  - all the (possibly consolidated) stm32 lib and usb example code

usb.c 
  - USB-specific hardware setup. Interrupts, clocks, etc. handling USB when 
    not "Attached". some low-level callbacks (low power mode, init, reset, 
    resume, etc).

usb_callbacks.c 
  - aka endpoints: handling data transfer when "Configured". calls out to 
    application specific callbacks (i.e. DFU).

usb_descriptor.c 
  - aka application descriptor; big static struct and callbacks for sending
    the descriptor. 

main.c
  - main loop and calling any hardware init stuff. timing hacks for EEPROM 
    writes not to block usb interrupts. logic to handle 2 second timeout then
    jump to user code. 

hardware.c
  - init routines to setup clocks, interrupts, also destructor functions.
    does not include USB stuff. EEPROM read/write functions.

dfu.c
  - mostly the giant FSM case switch, also some USB endpoint callbacks

## TODO

* pack the structs
* use sizeof() for usb application descriptor once structs are packed
