# Jetson_orin_modify_kernel
Process for modifying a Jetson Orin Nano Super  kernel

## Download and build latest Jetson Linux package (36.4.3)
tag = jetson_36.4.3 

// Latest Jetson Linux is at:
https://developer.nvidia.com/embedded/jetson-linux

## Get the Driver Package (BSP) from the DRIVERS section

wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.3/release/Jetson_Linux_r36.4.3_aarch64.tbz2

tar -xvf Jetson_Linux_r36.4.3_aarch64.tbz2

./source_sync.sh -k -t jetson_36.4.3

## Install the required build tools:

sudo apt update
sudo apt install git wget quilt build-essential bc libncurses5-dev libncursesw5-dev rsync


## Download and extract cross-compilation toolchain

mkdir ~/l4t-gcc
cd ~/l4t-gcc
wget -O toolchain.tar.xz https://developer.nvidia.com/embedded/jetson-linux/bootlin-toolchain-gcc-93
sudo tar -xf toolchain.tar.xz

## Set up environment variables
// The following export is only for building on the laptop host (Not the Jetson)

export CROSS_COMPILE=~/l4t-gcc/bin/aarch64-buildroot-linux-gnu-

export ARCH=arm64

export CONFIG_LOCALVERSION=-tegra

export KERNEL_OUT=~/kernel_out

export MODULES_OUT=~/modules_out

mkdir -p $KERNEL_OUT $MODULES_OUT


cd ~/Linux_for_Tegra/source/kernel/kernel-jammy-src

## Clear out directory

make O=$KERNEL_OUT mrproper

## Enable the correct module  (For example support for Geschwister Schneider USB/CAN devices)
// Also, be sure and set config Localversion to "-tegra"

make O=$KERNEL_OUT defconfig
make O=$KERNEL_OUT nconfig

## Now compile the kernel  NOTE: nproc = number of processors, which is 6 for the Orin Nano
make O=$KERNEL_OUT -j$(nproc)

## Now build the Modules

make O=$KERNEL_OUT modules -j$(nproc)

## Install the modules

make O=$KERNEL_OUT INSTALL_MOD_PATH=$MODULES_OUT modules_install

## Transfer and install the kernel and modules to the Jetson Orin Nano Super

scp $KERNEL_OUT/arch/arm64/boot/Image apineda@192.168.1.167:/tmp/
scp -r $MODULES_OUT/lib/modules/* apineda@192.168.1.167:/tmp/

## On the Jetson Orin Nano Super, install the new kernel and modules

sudo cp /tmp/Image /boot/Image
sudo cp -r /tmp/5.15* /lib/modules/
sudo depmod -a

## Update the bootloader configuration to point to the new kernel image
sudo vi /boot/extlinux/extlinux.conf

## Reboot and make sure the module is loaded properly
sudo reboot
lsmod | grep gr_usb

## If not loaded automatically, load it manually
sudo modprobe gr_usb









