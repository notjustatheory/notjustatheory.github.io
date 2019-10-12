---
title: "Building an Embdedd Linux System using Buildroot"
excerpt: "Lets build a full fladged embedded Linux system for Raspberry pi."
header:
  teaser: /assets/contents/embedded/buildroot_logo.jpg 
---
# **Introduction**

The goal of this blog is to describe the process of creating a base embedded Linux design upon which many other projects could be built. We seek a framework in which we can develop qt GUI programs. We want this Linux system to be lightweight and ready to be used for commercial purposes. It means that we want it to have the following characteristics: 



1. It should boot with a beautiful splash screen fitting the product.
2. It’s shut down/power up procedure should be controllable through an external command
3. When it’s shutting down, it must show the same splash screen as it showed when it was booting
4. For debugging and application development purposes, the sshd daemon should be running and configured appropriately 
5. Since it’s aimed to be used as an embedded system, it has to support a touch screen interface
6. The appropriate interfaces (i.e. UART , SPI) should be enabled
7. System’s IP Address must be static to avoid any ambiguity when developing applications

Satisfying the aforementioned criteria requires us to apply many configurations to the Buildroot standard settings. In the following, We go through the whole process step by step.


## Downloading the Buildroot

First we should obtain the Buildroot source code. For this purpose I’ve forked the current version of the buildroot and I build this project upon that. To download it you can  run the following commend in the working directory:


```
git clone https://github.com/Rooholla-Khorrambakht/buildroot.git
```


Or you can obtain it from the official repository:


```
git clone https://github.com/buildroot/buildroot
```


After downloading the source, we apply the base configuration for the raspberry pi with Qt support enabled:


```
make raspberrypi3_qt5we_defconfig
```


Now we should run the menuconfig tool and apply the required customizations:


```
make menuconfig
```



## General Configs:

There are some general configurations we need to do change such as the root password, the system banner, and the static IP address of the system. First we get the system banner and root password. To do this, navigate to  _System Configuration _and change the **System banner, System hostname, Root Password, Root filesystem overlay directories. **The last option should be set to a path on the development PC where the root file system files that We want to be added to the final image are stored. So go ahead and create this directory:


```
mkdir <where ever you want>/overlay 
```

![alt_text](/assets/contents/embedded/buildroot_pic1.jpg)


Next the maximum size of the image should be extended. For this, we should navigate to _Filesystem Images _and change the **exact size **option. 700 Mb is a good number.

![alt_text](/assets/contents/embedded/buildroot_pic2.jpg)


For setting a static IP address, the following structure should be defined  in the overlay folder:

**/etc/network/interfaces**

In this file, the following entries should be stored:


```
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

iface eth0 inet static
address 192.168.50.2
netmask 255.255.255.0
gateway 192.168.50.1
```


This is  where the static network IP is defined. Moreover, We want to explicitly tell the buildroot not to configure the **eth0** interface through DHCP. For this, we should navigate to **System Configuration **and clean the entry of the ** Network interface to configure through DHCP **parameter. 


## Adding Target Packages:


## SSH Server

From _Target Packages -> Networking applications **openssh**_ should be enabled. Then, another file must be added to the overlay for the openssh to allow root login sessions. For this purpose, a copied version of /etc/ssh/sshd_config with the line **PermitRootLogin yes **commented out should be added to the corresponding representative directory in the overlay folder: 


```
overlay/etc/ssh/sshd_config
```



## Enabling the X server

Qt requires some libraries and applications to be enabled to operate in xcb Mode. This requirements are documented in the [Here](https://doc.qt.io/Qt-5/linux-requirements.html). So, After enabling the X windowing system from _Target Packages -> Graphic libraries and applications (graphic/text) -> X.org, _the mentioned libraries and applications must be enabled:


### Servers



*   **_X.org ->  X11R7 Servers->xorg-server_**


### Libraries



*   **_X.org ->  X11R7 Libraries -> xcb-util-renderutil_**
*   **_X.org ->  X11R7 Libraries -> xcb-util-cursor_**
*   **_X.org ->  X11R7 Libraries -> xcb-util-keysyms_**
*   **_X.org ->  X11R7 Libraries -> xcb-util-wm_**
*   **_X.org ->  X11R7 Libraries ->libFS_**
*   **_X.org ->  X11R7 Libraries ->libICE_**
*   **_X.org ->  X11R7 Libraries -> libSM_**
*   **_X.org ->  X11R7 Libraries -> libXScrnSaver_**
*   **_X.org ->  X11R7 Libraries ->libXaw_**
*   **_X.org ->  X11R7 Libraries->libXcomposite_**
*   **_X.org ->  X11R7 Libraries->libXfont_**
*   **_X.org ->  X11R7 Libraries->libXtst_**


### Applications:



*   **_X.org ->  X11R7 Applications -> xauth_**
*   **_X.org ->  X11R7 Applications -> xinput_**


### Drivers:



*   **_X.org ->  X11R7 Drivers -> xf86-input-evdev_**
*   **_X.org ->  X11R7 Drivers -> xf86-input-mouse_**
*   **_X.org ->  X11R7 Drivers -> xf86-video-fbdev_**
*   **_X.org ->  X11R7 Drivers -> xf86-video-tslib_**


## Enabling The libTS

For calibrating resistive touchscreens, a handy library called tslib must be added to our system. This library is located at **Target Packages -> Libraries -> Hardware Handling -> tslib**.


## Adding The WiringPi

For Communicating with embedded microcontrollers and managing GPIOs, a good library called WiringPi exists. To enable it, we should navigate to **Target Packages -> Libraries -> Hardware Handling -> wiringpi **and enable this library. 


## Enabling the Qt Libraries:

There are some important qt libraries that do not get activated by the standard defconfig of the buildroot. Moreover, the native configuration of the buildroot uses the **eglfs **platform for rendering the graphics. However, in this system, we wish the X server to be our windowing system and we want Qt to use it. Therefore,, in addition to the extera application libraries, we have to enable the **xcb** support as well. In the following, the required packages and libraries are listed. From the **Target Packages -> Graphic libraries and applications (graphic/text) -> Qt5 **enable the following options:



1. **Qt53d**
2. **X.org XCB support**
3. **Qt5canvas3d**
4. **Qt5charts**
5. **Qt5enginio**
6. **Qt5graphicaleffects**
7. **Qt5imageformats**
8. **Qt5serialbus**

These Packages are the most frequently required libraries when developing an embedded GUI application using Qt. 


## Adding fbv Application

As mentioned before, we want the system to show us a splash screen when it shuts down. For this reason we need some kind of terminal based image viewer. **fbv** is an application that exists in the buildroot by default. So, to activate it we should navigate to **Target Packages -> Graphic libraries and applications (graphic/text) .**


## Compiling

At this point, All the required applications and libraries are activated so we are ready to issue the compile command. To do this, run the following in the buildroot directory:


```
make
```


After the compiling is done, the created image can be found at:


```
[buildroot dir]/output/images/sdcard.img
```


Now, we burn the SD card with this image using the dd command:


```
dd if=[buildroot dir]/output/images/sdcard.img of=/dev/<SD Card> status=progress
```



# Further Customization

So far, the we have downloaded and configured the build root. We have also compiled the base system. However, the created Linux system is not yet what we wanted it to be. Thus, we need to further customize the rootfs to meet our requirements. In the first step we need to create the boot logo and add it to the Linux source tree. 


## Custom Splash Screen

This is achieved through compiling a new kernel and importing  the custom boot logo into this newly created kernel. The process mentioned here is based on sources [Link1](https://developer.toradex.com/knowledge-base/splash-screen-linux), [Link2](http://www.rasplay.org/?p=6371).


### Installing the image editing tool and creating the logo

For editing and creating the logo image a cool software under linux is being used. First we install the software and the related dependencies that will be useful when creating our own images. 


```
sudo apt-get install gimp
sudo apt-get install tgif xfonts-100dpi xfonts-75dpi
```


After creating the logo, we should save it in a raw format with .ppm extension. To do this we should navigate to _File->Export. _There, we should add the .ppm extension to the file name and save it in the raw format:

![alt_text](images/Custom-Linux4.png "image_tooltip")



### Converting the picture to ASCI Format

After the logo is created, the ASCI raw format of it should be also generated. For this purpose the following commands should be executed:


```
$ ppmquant 224 toradexlogo_1024x600.ppm > toradexlogo_1024x600_224.ppm
pnmcolormap: making histogram...
pnmcolormap: Scanning image 0
pnmcolormap: 271527 colors so far
pnmcolormap: 271527 colors found
pnmcolormap: choosing 224 colors...
pnmremap: 224 colors found in colormap

$ pnmnoraw toradexlogo_1024x600_224.ppm > toradexlogo_1024x600_ascii_224.ppm
```



### Adding our picture to the Linux sources and compiling the kernel

After creating the ASCI raw image, We put it in the proper directory of the kernel source overwriting the default logo. 


```
cp toradexlogo_1024x600_ascii_224.ppm logo_linux_clut224.ppm
mv logo_linux_clut224.ppm ~/linux-toradex/drivers/video/logo/
```


Since we are using Buildroot as our build system, we should apply the image kernel that Buildroot has created automatically. This kernel is located in the following path:


```
[buildroot dir]/output/build/linux-<some long version number>/
```


Moreover, for cross compiling the kernel, we should use the cross compiler that the Buildroot has made for us. This cross compiler is located at:


```
[buildroot dir]/output/host/bin/
```


The suffix of this cross compiler is **arm-linux-. **Hence, the compile command would be:


```
make ARCH=arm CROSS_COMPILE=[buildroot dir]/output/host/bin/arm-linux- zImage
```


After the compiling is done, we should copy the newly created image and put it into the first partition of the SD card, replacing the old image. 


```
cp [linux source dir]/arch/arm/boot/zImage [sd card mount point]
```



### Splash screen at shutdown

For the splash screen to be displayed at shutdown, we should use the fbv tool to show our logo as soon as the shutdown procedure initiates. For this reason and other later usages, we should copy the inittab file from the image we just created. Being responsible for running things at power up and power down, the inittab is the config file used by busybox init program. This file is located at:


```
/etc/inittab
```


We should put this file in the corresponding directory in our overlay folder so that through later image creations, buildroot copies it into the rootfs. In dis file add the following entry:


```
::shutdown:/usr/bin/fbv /opt/logo.jpg &
```


We note that the logo.jpg should be present in the /opt directory of the rootfs. Therefore, we must put it into the overlay/opt directory of the Buildroot system so that it would be copied later. 


## Making the Linux kernel go quiet at boot time

By default, the Linux kernel prints out many debug messages and a blinking cursor when booting. For disabling them, we should modify the kernel command at the cmdline.txt:


```
root=/dev/mmcblk0p2 rootfstype=ext4 rootwait console=tty3 quiet vt.global_cursor_default=0
```



## Creating an Application starter Script

For the user Qt app to be executed at startup, we need to create a script where we define the appropriate local variables, run the X server and finally execute the application binary. This script is as follows:


<table>
  <tr>
   <td><code>killall ploter_project \
killall X \
export QMLSCENE_DEVICE=softwarecontext #Use the software renderer for qml \
export DISPLAY=:0 #Assign the Display variable for the X server \
export QT_QPA_PLATFORM=xcb #Qt should use the X server for drawing the graphics \
X -nocursor & # start the X server \
/opt/application_binary #start the user application</code>
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
</table>


Like before, this script alongside the application binary should be placed at the overlay folder **overlay/opt. **Moreover, for running at startup, the following line has to be added to the **/etc/inittab** file:


```
::respawn:/opt/appStart.sh
```



## Setting Up the Touchscreen


### Touchscreen device rules

By default touchscreens are known to the system as an input device, **eventx,** located in the **/dev/input** directory. Considering the fact that number x is assigned based on the number of input devices attached to the system, it is prone to changes. Here, we want to add a device rule that recognizes the touch screen chipset an creates a specific symbolic link representing it. This can be done by exploiting the capabilities offered by udev system. We should create a device rule with following line stored in it:


```
SUBSYSTEM=="input", KERNEL=="event[0-9]*", ATTRS{name}=="ADS7846 Touchscreen", SYMLINK+="input/touchscreen"
```


This file should be copied into the address **/etc/udev/rules.d/. **Therefore,  we put it into a corresponding directory of the overlay folder for the Buildroot to copy it into the rootfs.

Moreover, for the touchscreen to be enabled the appropriate device tree overlay should be activated by putting the following entry in the config.txt file:


```
dtoverlay=ads7846,cs=1,penirq=25,penirq_pull=2,speed=50000,keep_vref_on=0,swapxy=0,pmax=255,xohms=150,xmin=200,xmax=39$
```


This command is specific to displays like 5 inch waveshare lcd that use ads7846 compatible touchscreen drivers.


### Calibrating the Touchscreen

Now, we should run the ts_calibrate command and follow the required steps. If successful, the parameter files will be stored in the appropriate location of the rootfs. We can then copy that files into the overlay folder so that we would not have to re do all this process again for the new image. The final step is to create a uinput device that emulates a touchscreen which reads the calibrated values. To do this reason, **ts_uinput -v -d **deamon should be called at startup. This can be done by putting the following line into the **/etc/inittab **file.


```
::sysinit:/sbin/modprobe uinput
::sysinit:/usr/bin/ts_uinput -d -v
```


The first line in this listing loads the **uinput** module into the kernel which is a prerequisite for the  **ts_uinput** command. 

Two good sources regarding tslib are [Here](https://www.impulseadventure.com/elec/rpi-install-tslib.html) and [Here](https://github.com/raysan5/raylib/wiki/Install-and-configure-Touchscreen-Drivers-(RPi)).


### 


### Explicitly telling the X server to use the calibrated touchscreen

By default, the X server reads all the input devices in the /dev/input. Here we wish to tell the X server to use only the calibrated touchscreen uinput device. To do this, we need to create a new file called **80-tslib.conf **in the **/usr/share/X11/xorg.conf.d** directory. The number 80 in the filename is arbitrary and indicates the relative order of applying the configuration. The content of this newly created file should be as follows:


```
Section "InputClass"
    Identifier "tslib touchscreen catchall"
    MatchIsTouchscreen "on"
    MatchDevicePath "/dev/input/*"
    Driver "tslib"
EndSection
```



## Setting the Display resolution

A fixed resolution can be set for the specific type of lcd we are using . To do this, we should add the following lines to the config.txt file:


```
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=1
hdmi_mode=87
hdmi_cvt 800 480 60 6 0 0 0
```



## Enabling the required interfaces

For many embedded linux applications, SPI, UART and I2C are integral interfaces that almost every system designer would use to connect the microcontroller to the raspberry pi. Activating these interfaces for the raspberry pi requires us to add the following lines to the config,txt:


```
dtparam=i2c_arm=on
dtparam=spi=on
enable_uart=1
dtoverlay=spi1-3cs
```



## Automating the config.txt and cmdline.txt modification at build time

So far, the required modifications to the cmdline.txt and config.txt had to be applied manually. In this section this process is automated by modifying some Buildroot specific files and creating some place in which we can write our modifications. When the build process is done, Buildroot runs a particular script called post-image.sh.The arguments passed to this script are set in the menuconfig. What this script does is to put the constituents of the FAT32 SD card drive into the appropriate section of the created image. Here we want to add an extra argument to this script that reads some files and puts their contents into the cmdline.txt and config.txt. We want this extra arguments to be **--apply-custom-configs**. In the listing below, the content of the post-image.sh is shown.


```
#!/bin/bash

set -e

BOARD_DIR="$(dirname $0)"
BOARD_NAME="$(basename ${BOARD_DIR})"
GENIMAGE_CFG="${BOARD_DIR}/genimage-${BOARD_NAME}.cfg"
GENIMAGE_TMP="${BUILD_DIR}/genimage.tmp"

for arg in "$@"
do
    case "${arg}" in
     --add-pi3-miniuart-bt-overlay)
     if ! grep -qE '^dtoverlay=' "${BINARIES_DIR}/rpi-firmware/config.txt"; then
       echo "Adding 'dtoverlay=pi3-miniuart-bt' to config.txt (fixes ttyAMA0 serial console)."
       cat << __EOF__ >> "${BINARIES_DIR}/rpi-firmware/config.txt"
# fixes rpi3 ttyAMA0 serial console
dtoverlay=pi3-miniuart-bt
__EOF__
     fi
     ;;
     --aarch64)
     # Run a 64bits kernel (armv8)
     sed -e '/^kernel=/s,=.*,=Image,' -i "${BINARIES_DIR}/rpi-firmware/config.txt"
     if ! grep -qE '^arm_64bit=1' "${BINARIES_DIR}/rpi-firmware/config.txt"; then
       cat << __EOF__ >> "${BINARIES_DIR}/rpi-firmware/config.txt"
# enable 64bits support
arm_64bit=1
__EOF__
     fi

     # Enable uart console
     if ! grep -qE '^enable_uart=1' "${BINARIES_DIR}/rpi-firmware/config.txt"; then
       cat << __EOF__ >> "${BINARIES_DIR}/rpi-firmware/config.txt"
# enable rpi3 ttyS0 serial console
enable_uart=1
__EOF__
     fi
     ;;
     --gpu_mem_256=*|--gpu_mem_512=*|--gpu_mem_1024=*)
     # Set GPU memory
     gpu_mem="${arg:2}"
     sed -e "/^${gpu_mem%=*}=/s,=.*,=${gpu_mem##*=}," -i "${BINARIES_DIR}/rpi-firmware/config.txt"
     ;;
     --apply-custom-config) #Our Custom configuration argument
     cat ${BOARD_DIR}/custom_config.txt >> ${BINARIES_DIR}/rpi-firmware/config.txt
              cat ${BOARD_DIR}/custom_cmdline.txt > ${BINARIES_DIR}/rpi-firmware/cmdline.txt

    esac

done

rm -rf "${GENIMAGE_TMP}"

genimage                        \
    --rootpath "${TARGET_DIR}"  \
    --tmppath "${GENIMAGE_TMP}" \
    --inputpath "${BINARIES_DIR}"  \
    --outputpath "${BINARIES_DIR}" \
    --config "${GENIMAGE_CFG}"

exit $?
```


As it can be seen, there is an extra option added to the case statement. Named **--apply-custom-conig, **this section adds the contents of files **custom_config.txt **and **custom-cmdline.txt** to the **config.txt **and **cmdline.txt** respectively. We should put this custom files the at the same place where the script is added. This location would be:


```
[buildroot dir]/boards/raspberrypi3/
```


Finally, We need to tell the Buildroot to pass our newly created argument to this script. To do this we should navigate to **System configurations ->Extra arguments passed to custom scripts **in the menuconfig and add **--apply-custom-conig. **


## Remote Halt Command

For Embedded system applications we often want be able to control the power state of the system using an external digital signal. As Embedded Linux systems are generally vulnerable to accidental shutdowns, a proper mechanism should be in place. For this reason, a custom daemon has be developed that listens to an external GPIO and issued a halt command on a detection of a falling edge. This daemon should be compiled using the Buildroot cross compiler and the produced binary should be placed in the **/opt **directory of the system. Moreover, this daemon has to be called on startup by adding the following line in the inittab file:


```
::respawn:/opt/halt_deamon
```


# Link to Qt Creator

Now that the image is ready and the build system is in place, we can use the qt creator software for developing our applications. For this purpose there is a great tutorial that explains how to link Buildroot to Qt Creator. This is located [Here](https://github.com/pbouda/buildroot-qt-dev/blob/master/doc/qtcreator.md).
