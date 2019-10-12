title: "Getting into Petalinux and Pynq (Part1)"
excerpt: "Pynq and PetaLinux for Xilinx SOCs."
header:
  teaser: /assets/contents/embedded/petalinux_logo.png 

# Introduction


## Petalinux 

Petalinux is a tool suite developed by Xilinx. Built on top of the Yocto project, this tool makes the process of creating  linux systems for Xilinx SOCs extremely easy and systematic. This tool is in tight harmony with the Vivado software sutit and accepts customly created SOC designs form this software. From there, the toolchain creates the device tree according to the particular design in the Vivado project and produces fully functional linux images for it. 


## Pynq

Pynq is linux distribution created on top of kernels produced by the Petalinux. This platform aims to make the process of exploiting the FPGA part of the SOC easier by introducing the concept of overlays. Moreover, this platform is centered around the python programming language and inherits all the lovely things about it. Exploiting the custom linux drivers by Xilinx, the pynq platform introduces easy and fast methods for connecting to the customly created hardware accelerators located in the FPGA part of the SOC. In fact, the mechanism of loading custom hardware designs on to the FPGA on the go is just like including a library in the conventional programming paradigm. This process is coined as overlay loading.


## What is the purpose of this blog?

The purpose of this document is to present the process of building a Pynq image for a custom Zynq board. Next, the process of developing designs based on pynq is illustrated through two example projects. The first project is a simple hardware accelerator multiplying two matrices and the second project illustrates using pynq for hardware accelerated image processing tasks.


# Step 1, Petalinux BSP

For creating a Pynq image for our custom board, first we need to build a PetaLinux BSP for it. This BSP will be used as we create the Pynq image and based on that, Pynq build system will create a linux kernel compatible with the hardware we defined in the BSP. But why? As we already know, unlike a regular SOC which is immutable and fixed , the Xilinx Hybrid FPGA can have any kind of configurations and custom  hardwares in it. For example, the PS (Processing system part of the FPGA where the ARM cores are located) can have many kind of configurations. Configurations such as, what kinds of AXI interfaces are should be activated, what clocks are applied to the PL (Programmable Logic of the FPGA part of the SOC) and many more. What is Important to note is that when we create a Pynq image, our Pynq overlays should also use the same configs for their PS as the configs upon which the image was built. In other words, when the configurations of the PS are set at boot time, they can not be changed while the Linux is up and if we apply an overlay with a different PS settings, it’s quite likely for the system to crash. Thus, as a first step toward creating a Pynq image for our board, we should build a Vivado Project whose PS settings are set in accordance to the hardware overlays that are to be loaded during runtime when the system is functional and booted. One good starting point for looking up this settings is the base overlay of a standard Pynq image. For example, the PS settings of the Pynq-Z1 board would be a good template for any board that uses a Zynq7020 chipset.

However in general, the minimal resources needed for a Linux based system to boot on a Zynq 7000 is a TTC timer, DDR memory and a non-volatile storage(SD Card). 


## Vivado Hardware Project for Z-Turn 

In this step, I’ll explain the process of building a basic hardware project which is suitable for a Linux based system and is compatible with the parameters required by a custom board called [Z-Turn](http://www.myirtech.com/list.asp?id=502). First, for this board to be recognized by the Vivado software, some board definition files should be copied into specific locations of the Vivado installation directory. We need to download this board definition files  from this [Github repository](https://github.com/q3k/zturn-stuff). Next, based on the instructions given in this repository we should copy the files we just downloaded to the appropriate locations. Now that we have added the board, we begin by creating a project in Vivado. For this project we choose the board we just added as the hardware:

![](/assets/contents/embedded/petalinux_pic1.jpg)


Then, we create a block design and add a zynq processing system into it:

![](/assets/contents/embedded/petalinux_pic2.jpg)


At this point, the Vivado software is telling us that there are some automatic configurations that it can apply to the current design. This configurations are the board specific presets and the connection of the fixed IOs to the external pins. We accept this offer running the block automation:

![](/assets/contents/embedded/petalinux_pic3.jpg)


As a result of this configuration, the parameters of the DDR memory and other board specific settings will get applied to our design. 

![](/assets/contents/embedded/petalinux_pic4.jpg)


As mentioned before, this settings will be fixed later of as we develop our custom overlays for our Pynq system. We can export create the HDL wrapper and export the hardware as it is but, it is a good idea to stay compatible with the standard design of the Pynq Z1 board. To do this, we can download the Pynq-Z1 presets from [Here](https://reference.digilentinc.com/_media/reference/programmable-logic/pynq-z1/pynq_revc.zip) and create a new project and apply those presets to the processing system. Then we can mimic the important settings and apply them to our own project. This settings include:



1. Enabling the AXI interfaces 
2. Enabling the PL-PS interrupt lines
3. Enabling the PL clocks and setting their values
4. Enabling the GPIO EMIO pins 

After we do this, our PS block will look like this:

![](/assets/contents/embedded/petalinux_pic5.jpg)


Moreover, there are some board specific configurations that should be applied. First, in the Z-Turn board, the I2C0 peripheral is mapped to the FPGA pins and is responsible for managing the on-board chips like HDMI transmitter and I2C sensors. Second, the reset signals of the Ethernet-PHY and HDMI transmitter is controlled by MIO51 pin. So we must enable this reset in the MIO configuration section of the PS configuration wizard. In summary, we have to combine our board specific settings with the ones for the standard Z1 board. After these configurations are applied we should create an HDL wrapper for our block design. Next we Create the Bitstream  and export our hardware to be used by the Petalinux later. To do this, we need to navigate to **File -> export -> hardware** and select an appropriate location. The result will be a .**hdf** file which will later be used for creating a petalinux project. The reader can acquire this basic project form author’s github repository.


## PetaLinux, A base linux kernel for Pynq

In this step, we use the Vivado project we created in the previous section to create a linux board support package (BSP) using the Xilinx Petalinux tools. To do this,  First We have to create a simple functional linux image using the petalinux tools. It’s important to note that the process that is being described here is for Zynq 7000 family. For Ultrascale and microblaze the process is slightly different. 


### Setup the Environment

When the petalinux is successfully installed, we need to source a shell script in the installation directory in order to use the toolchain:


```
$ source <path-to-installed-PetaLinux>/settings.sh
```


**Note**: It’s important to note that petalinux tools requires the python 2.7 to be our default python interpreter.

Now if we check $PETALINUX environmental variable we should get the installation directory of the petalinux tool in our system:


```
$ echo $PETALINUX
/opt/pkg/petalinux
```


In addition to petalinux, the vivado and SDK  scripts  should also be sourced:


```
$ source <path-to-installed-Xilinx-Vivado>/settings64.sh
$ source <path-to-installed-Xilinx-SDK>/settings64.sh
```



### Crating a Petalinux Project

First, we need to create a base project. To do this we should run the following command:


```
$ petalinux-create --type project --template zynq --name plnx_project
```


This will create a project named plnx_project in the directory in which we are. 


### Importing the hardware configuration

Now, we need to import the hardware we created using the Vivado design suite. To do this, First we navigate to our petalinux project directory and then run the following command:


```
$ cd <plnx-proj-root>
$ petalinux-config --get-hw-description=<path-to-directory-containing-hardware description-file>
```


As the input to this command we need to give the address to the **.hdf** file we created using Vivado in the previous section. As a result of this command, a menuconfig window will appear:

![](/assets/contents/embedded/petalinux_pic6.jpg)

First, we need to ensure that the **Subsystem Auto Hardware Settings** is enabled. Using this option, we can set system wide configurations like Ethernet, Console, memory range and many more. For example, here we set the Ethernet IP configs to the static mode and assign the static IP address. So we  navigate to **Subsystem Auto Hardware Settings** **-> Ethernet Settings **and disable the **Obtain IP Address Automatically. **Doing so, enables the static ip fields which we assign based our own preferences. Finally, by pressing **Esc** key twice, we save the configurations and exit the menu. 

**Note:** in case that we want to relaunch this menus later we can run  the petalinux-config again. Moreover, we can specifically set the configurations of system components using the -c option:


```
$ petalinux-configure -c <component> 
```


For the components, we can use any of the following:



1. **kernel : **linux kernel menuconfig 
2. **rootfs : **root file system packages configs 
3. **u-boot: **u-boot menuconfig 
4. **bootloader: **bootloader menuconfig
5. **pmufw: **Zynq Ultrascale power management unit menuconfig 
6. **device-tree: **device tree configs


### Building the Project

Now that the project is created and the hardware is imported to it we can build the petalinux image by running the following command:


```
$ cd <plnx-proj-root>
$ petalinux-build
```


After the build process is done, the created images can be found at **<plnx-proj-root>/images/linux/ **directory. 


### Generating the boot image 

Now that the project is built, we should create the bootable image. To do this, we run the following command:


```
$ petalinux-package --boot --fsbl <FSBL image> --fpga <FPGA bitstream> --u-boot
```


We need to set the <FPGA bitstream> parameter to the address of the bitstream we created using our vivado project. Furthermore, the fsbl file can be found at the directory the images were created. As a result of running this command, the bootable image will be created for us and we can find it in the **<plnx-proj-root>/images/linux/** directory. 


### Booting the image 

First, we package the images we just created using the following command:


```
$ petalinux-package --prebuilt --fpga <FPGA bitstream>
```


After we do this, we can boot our image through a variety of ways. Here, we want to use a simple SD card to boot the board and we leave the other method to a later time. To boot the image, we need to copy the two **BOOT.BIN** and **image.ub** files from **<plnx-proj-root>/images/linux/ **directory into a FAT32 formatted SD card. Then, we put the SD card on the board and set the jumpers so that zynq boots the system from the SD interface. If every thing has been done correctly, we should be seeing the kernel messages from the uart console. 


### Creating a BSP for Custom board

Now that we have created a working petalinux image and a base hardware project, we want to pack them all in a BSP so that we can use it later, when we create the Pynq image. Importantly not only is a BSP useful for creating the Pynq, but also we can use it to share our project with others as well. To make this, all we need to do is using the following command:


```
$ petalinux-package --bsp -p <plnx-proj-root> \ --hwsource <hw-project-root>
--output MY.BSP
```


In the above command, we need to supply the addresses of our vivado and petalinux projects.


# Step 2, Using the Custom BSP for Making the Pynq

In this step we want to use the BSP we created in the previous section in order to make our custom Pynq linux system. But, before doing anything, we need to clone the Pynq repository into our system:


```
git clone https://github.com/Xilinx/PYNQ.git
```


Now, the first thing we need to do is running a script that creates the build environment for us. To do this, we source that script as follows:


```
cd pynq
source sdbuild/scripts/setup_host.sh
```


This will download and install some important tools which are required for building the image. Moreover, for building the image, pynq requires the vivado, SDK and Petalinux to be recognized in the terminal. Therefore, we also need to source the setup scripts for these softwares:


```
source <Vivado installation directory>/settings64.sh
source <SDK installation directory>/settings64.sh
source <Petalinux installation directory>/settings.sh
```


Now, we should add our own board to the boards folder. To do this, we use the PYNQ-Z1 board as a template:


```
cd boards/
cp -r PYNQ-Z1 Z-TURN
cd Z-TURN
```


At this step, we need to customize the this template to be fit to our own custom board. So we change the name of the **.spec** file to our own board and open it using an editor to put in it our custom configurations:


```
mv PYNQ-Z1.spec Z-TURN.spec
vim Z-TURN.spec
```


The current content of the .spec file is:


```
ARCH_Pynq-Z1 := arm
BSP_Pynq-Z1 :=
BITSTREAM_Pynq-Z1 := base/base.bit
STAGE4_PACKAGES_Pynq-Z1 := pynq boot_leds ethernet
```


We modify the file so that it would look like this:


```
ARCH_Z-TURN := arm
BSP_Z-TURN := Z_TURN.bsp
BITSTREAM_Z-TURN := base/base.bit
STAGE4_PACKAGES_Z-TURN := pynq ethernet
```


Here, we have disabled the boot_leds service by removing the  boot_leds entry from the STAGE4_PACKAGES_Z_TURN option. Furthermore, for the BSP_Z-TURN we have supplied the address for our BSP that we created from the previous section (This file should be later copied into the Z-Turn folder we just created). We save the changes and exit. 

Next, we need to change the base.bit file. The current file located in base/ directory is specific to the PYNQ-Z1 board. So we must replace it with the bit file we created using for our own board:


```
cp <path to the newly created bit file>  base/base.bit
```


Moreover, we should copy in the BSP file as well:


```
cp <path to the BSP file>  ./
```


Note that the name for the BSP file should be in accordance to the name we entered in the spec file.

Finally to build the Image, we navigate to sdbuild directory of the pynq repository and run the make command as follows:


```
cd <Our Pynq repository>/sdbuild
make BOARDS=Z-TURN
```


This will initiate the build process and it would take a long time to complete. At the end, when the build is finished, our newly created Pynq image will be available at … . So we can simply burn our SD card using this image and we should be ready to go. Our Custom Pynq system is now functional. 


