# Raspberry Pi 400

![SystemReady-ES Certified](../../_assets/systemready_icons/es.png)

What is needed:
- [Raspberry Pi 4 400](https://www.raspberrypi.org/products/raspberry-pi-400-unit/)
- USB-C power supply
- MicroSD card or USB drive if using [USB bootloader](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/msd.md) (at least 16MB)
- If using microSD: A USB card reader

# Making the board SystemReady
1) Download the latest Raspberry Pi 4 UEFI Firmware image from https://github.com/pftf/RPi4/releases.

2) Format MicroSD/USB to FAT32.

3) Extract the zip file downloaded in step 1) to the newly formatted
   MicroSD/USB.

4) Properly eject the MicroSD/USB to ensure it has finished writing, and insert
   it into the board.

5) The board is now ready to boot SystemReady compatible operating systems from
   a SD Card or USB. Note: The SD/USB created in this guide must remain, the OS
   installer must be on a *second* drive, and the OS then be installed onto a
   *third* drive.

