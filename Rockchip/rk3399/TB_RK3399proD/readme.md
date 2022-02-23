# ToyBrick RK3399ProD

![SystemReady-IR Certified](/_assets/systemready_icons/ir.png)

Certification:
[SystemReady IR](https://armkeil.blob.core.windows.net/developer/Files/pdf/certificate-list/arm-systemready-ir-certification-rockchip.pdf)

Certification Errata:
[SystemReady IR](https://armkeil.blob.core.windows.net/developer/Files/pdf/certificate-list/arm-systemready-errata-document-rockchip-tb-rk3399pro-414.pdf)


**What is needed:**
- ToyBrick RK3399 ProD AI Development Kit
- Power supply
- USB type-C connector to interface with host machine
- Micro USB to USB connector for UART connection to Host machine 
- Host Machine (x86_64/amd64) running a recent Ubuntu Distro (tested with 20.04)



# 1. Prepare Host machine
The following package installations and steps are required to be completed to build and install the firmware binaries.

```
sudo apt-get install -y cmake python3 python-is-python3 git gnupg bison flex
sudo apt-get install -y libssl-dev device-tree-compiler

# Configure git with your details
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

# Install repo
#   Option 1 - package manager
sudo apt-get install repo

##   Option 2 - manual download
##  curl https://storage.googleapis.com/git-repo-downloads/repo > /opt/repo
##  chmod +x /opt/repo
##  export PATH=$PATH:/opt
```


# 2. Clone the repositories
```
mkdir <New_Dir>
cd <New_Dir>
repo init -u https://gitlab.arm.com/systemready/firmware-build/rk3399-manifest -m TB-RK3399proD.xml
repo sync -j4 --no-clone-bundle
```

# 3. Get the Toolchains
If you dont have an aarch64-linux-gnu cross compiler already installed on your host machine, there are 2 ways of suppling the toolkits required to build the firmware and U-Boot binaries.

* Option 1: Package install the cross-compilers on the host machine OS.
* Option 2: Use the toolchain installer thats provided by the makefile

## Option 1: PREFERRED

```
sudo apt-get install gcc-aarch64-linux-gnu gcc-arm-none-eabi
which aarch64-linux-gnu-gcc
which arm-linux-gnu-gcc
```
Copy the path reported by the above command (excluding the tailing `bin/aarch64-linux-gnu-gcc`) and paste it after the command below:

```
export AARCH64_PATH=
```
For example:

```
$ which aarch64-linux-gnu-gcc
/usr/bin/aarch64-linux-gnu-gcc
$ export AARCH64_PATH=/usr
$ echo $AARCH64_PATH
/usr
```

## Option 2: ALTERNATIVE (skip this if using option 1)

```
cd build/
make -j2 toolchains
```

# 4. Build U-Boot binaries for target
```
cd build/
make -j `nproc`
```
Amongst other things the above command will generate the following files of interest in the `../out/bin/u-boot/` directory:

```
u-boot.itb
idbloader.img
```
Check the above by running:
```
ls -ltr ../out/bin/u-boot/
```

# 5. Building the Loader binaries
```
cd ../rkbin
./tools/boot_merger  ./RKBOOT/RK3399PROMINIALL.ini
```
The above command will generate the miniloader file: eg. `rk3399pro_loader_v1.25.126.bin`

# 6. Flashing the Loader binaries and Firmware

## 6-a: Preparing the host-machine
To succesfully detect and flash the board the following packages are required to be installed on the host machine

```
sudo apt-get install pkg-config libudev-dev libusb-1.0-0-dev libusb-1.0
```


## 6-b: Put the board in MaskROM mode
1. Power-off the board
2. Keep the Maskrom button pressed for 10 seconds
3. Power on the board (either by long-pressing the power button OR plugging in the Power connector)
4. Release maskrom button after 5 seconds
5. The device should now be in Maskrom mode ready for flashing
6. Check if the board has entered MaskROM mode by running `lsusb` and checking if the following device is detected:

```
Fuzhou Rockchip Electronics Company RK3399 in Mask ROM mode
```


### 6-c: Writing the binaries to device
```
sudo ./tools/rkdeveloptool db rk3399pro_loader_v1.25.126.bin && sleep 2
sudo ./tools/rkdeveloptool wl 0x40 ../out/bin/u-boot/idbloader.img && sleep 2
sudo ./tools/rkdeveloptool wl 0x4000 ../out/bin/u-boot/u-boot.itb && sleep 2
sudo ./tools/rkdeveloptool rd
```



The TB_RK3399ProD board should restart and boot into u-boot. If there is a pre-installed distro available on the on-board flash then the system will boot into it.


# Booting standard distros on external storage (USB)

If you have a bootable storage device with a Linux distro installed, connected to USB the board may not automatically boot into it. 

`
Note: 
HDMI/DP display is currently not supported in this firmware image. So to interact with the board ensure you are connected via UART over a micro-USB cable.
The default baudrate for the board is 1500000 (1.5M) 
`

You have 2 options to boot force USB boot: 

* Option1: Manual boot device selection: 
   1. Interrupt u-boot by hitting any key
   2. Run the command: 
   `run usb_boot`

* Option2: Update U-boot env variables 
   1. Interrupt u-boot by hitting any key
   2. Update the *boot* environment variables to either change the boot order or override the "boot" variable to "usb_boot"

This should start booting the linux kernel on the external uSDCard/USB drive. 

```
IMPORTANT: 

The ToyBrick RK3399ProD has a default UART baudrate of 1.5M (1500000). 
This might be changed by the external Distro as it boots.

Once you select the boot device in u-boot and it starts booting the standard distro it might look like it hangs.

This would indicate that the baudrate has changed and you will have to kill the terminal and re-connect via UART 
with baudate of 115200 to see the login prompt.

```


