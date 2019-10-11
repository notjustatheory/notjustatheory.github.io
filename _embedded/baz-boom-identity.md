---
title: "A Quick Hands-on Device trees for Embedded Linux applications"
excerpt: "For defining the hardware structure of an embedded system, We must know about device trees. In this blog, I document what I learn about this."
header:
  image: /assets/images/unsplash-gallery-image-1.jpg
  teaser: assets/images/unsplash-gallery-image-1-th.jpg
---
##Introduction 
The power of linux , as in an embedded linux context, comes from the huge pool of open source codes that are freely available to the developer. When we say embedded linux we are often thinking about a tiny computer which is  connected to a bunch of sensors, actuators and other kinds of hardware. Having their own mechanisms and operation principles, each of these devices require a driver in order to operate properly. In a conventional micro controller based system, all of these drivers have to be hand crafted and ported by the developer which introduced a huge load to the development process. However, Linux provides a global standard that makes things much easier. In Linux, the drivers act as an abstractor layer between the user of the device. In fact in Linux, all the device are presented to the user as files through which user can interact with the device and exchange data with it. 
Linux device drivers for thousands of devices are freely available on the net. So when we develop an embedded linux system, we seldom need to write any drivers and we can easily use the existing  ones on the net. However, in order to do this, we need to know how to tell Linux which hardwares are connected to it and it what ways. In other words, as the linux boots, it needs to know what devices there are  so that it knows what drivers to invoke. 
This task of defining the hardware structure of the system is done by file called device tree bob. The bootloader of an embedded linux system ( for example U-BOOT) passes this file to the kernel as it boots and through this linux knows what types of hardware and system structure it’s dealing with. This binary .dtb file is compiled from .dts source in which the structure of the system is defined in tree like data structure with a special syntax. 
Thus, the process of adding a device to the embedded linux system is as follows:
-If the driver is not included in the kernel as a module or built in, compile the kernel with the required driver enabled.
-Modify the default device tree source file of the particular board or SOC and add the new device to it.
-Compile the device tree using the device tree compiler (dtc) and create the device tree binary file (dtb)
-Replace the old dtb with the new one and linux should now recognize the new hardware as it boots
In this document, I intend to write a short tutorial on how syntax is and how we can add a new driver to our system. 
##The syntax and basic structures
In one picture, the basic syntax can be illustrated as follows


