# PINE64 ROCKPro64

![SystemReady-IR Certified](/_assets/systemready_icons/ir.png)

Certification:
[SystemReady IR](https://armkeil.blob.core.windows.net/developer/Files/pdf/certificate-list/arm-systemready-ir-certification-pine64.pdf)

Certification Errata:
[SystemReady IR](https://www.arm.com/-/media/Files/pdf/certificate-list/arm-systemready-errata-document-pine64-rockpro64-412)


**What is needed:**
- [PINE64 ROCKPro64](https://www.pine64.org/rockpro64/)
- Power Supply
- MicroSD card - minimum of 2GiB
- MicroSD card reader
- Access to a linux (virtual) machine

# Building the firmware

Firstly, ensure `repo` is installed on your system: https://source.android.com/setup/build/downloading

Create a new directory and `cd` into it
```
$ mkdir rockpro64_firmware
$ cd rockpro64_firmware
```

Clone the repositories
```
$ repo init -u https://gitlab.arm.com/systemready/firmware-build/rk3399-manifest -m RockPro64.xml -b refs/tags/rockpro64-21.09
$ repo sync -j4 --no-clone-bundle#
$ cd build
```

If you don't have an `aarch64` toolchains installed, run the following
```
$ make -j2 toolchains
```

Finally, build the firmware
```
$ make -j `nproc`
```

# Installing the firmware to the microSD card

Firstly identify the disk you will be writing to (For example using `lsblk`) and set it to a environment variable
```
export DISK=/dev/sdd
```

Next ensure you have a fresh start by filling the SD card with zeros:
```
$ sudo dd bs=4M if=/dev/zero of=$DISK oflag=sync
```

## Creating the partition table

Begin by creating a GPT partition table using `gdisk`
```
$ sudo gdisk $DISK
```

Create a new partition table
```
Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y
```

Now that you have created a fresh GPT partition table, you need to create a "protective" partition.
This partition will prevent the boot loaders you will write later on from being overwritten.
```
Command (? for help): n
Partition number (1-128, default 1): 1
First sector (34-30277598, default = 2048) or {+-}size{KMGTP}: 34
Information: Moved requested sector from 34 to 2048 in
order to align on 2048-sector boundaries.
Use 'l' on the experts' menu to adjust alignment
Last sector (2048-30277598, default = 30277598) or {+-}size{KMGTP}: 34816
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef02
Changed type of partition to 'BIOS boot partition'
```

Next, create and `EFI` partition for saving EFI configuration
```
Command (? for help): n
Partition number (2-128, default 2): 2
First sector (34-30277598, default = 36864) or {+-}size{KMGTP}: 34817
Information: Moved requested sector from 34817 to 36864 in
order to align on 2048-sector boundaries.
Use 'l' on the experts' menu to adjust alignment
Last sector (36864-30277598, default = 30277598) or {+-}size{KMGTP}: 2136964
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI system partition'
```

Before writing the changes to the SD card, check it has been setup correctly. The output should look similar to this:
```
Command (? for help): p
Disk /dev/sdd: 30277632 sectors, 14.4 GiB
Model: MassStorageClass
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): EA991EB6-05B5-4EDB-A235-07FEDC3707E0
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 30277598
Partitions will be aligned on 2048-sector boundaries
Total free space is 28144695 sectors (13.4 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048           34816   16.0 MiB    EF02  BIOS boot partition
   2           36864         2136964   1.0 GiB     EF00  EFI system partition

```

Next, Write the partition table to disk
```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdd.
The operation has completed successfully.
```

## Creating a FAT32 file system on the `EFI` partition

In order to be able to save EFI configuration, create a FAT32 file system on the newly created `EFI` partition
```
$ sudo mkfs -t vfat "${DISK}2"
```

## Copying the firmware to the micro SD card
Finally, copy over the firmware you built previously
```
$ sudo dd if=../out/bin/u-boot/idbloader.img seek=64    of=$DISK
$ sudo dd if=../out/bin/u-boot/u-boot.itb    seek=16384 of=$DISK
$ sync
```