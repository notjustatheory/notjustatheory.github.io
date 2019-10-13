---
title: "The simplest linux Framebuffer for ZYNQ FPGAs (Part1)"
excerpt: "How to get the video working with the Linux."
header:
  teaser: /assets/contents/embedded/zynqfb_logo.png 
---
# Introduction 

One of the greatest advantages of designing image processing systems with FPGAs is that we can create custom image processing IPs that can outperform even best CPUs in terms of performance to power ratio. Recently, with the advent of Hybrid FPGAs, we can use the combined powers of ARM CPUs and FPGA fabrics in Our Computing system. Indeed, we can leverage the power of CPU for the algorithmic tasks and the parallelizable nature of the FPGA for the computational intensive processes of the system. Not only that, but also we can exploit the huge ecosystem of Linux by porting  it to the CPU section of the Hybrid FPGA while we can seamlessly  link it to the custom accelerators that we have designed on the FPGA part of the chip.  This new paradigm of designing heterogeneous processing systems has expedited  the development process and reduced the final cost of the system. 

This document extends the previous tutorial  where I explained how we can create an Image processing pipeline on the FPGA part of a Zynq SOC and how we should link it to the CPU through AXI interfaces. In that tutorial, a bare metal software was being executed on the CPU that was only responsible for configuring the system and assigning the frame buffers to the pipeline. 

In This document, we take one step further and investigates the ways through which we can employ a Linux operating system on the CPU to control the image pipeline. Here, the image IO will be visible in Linux as frame buffer (** /dev/fbx **) devices. However, this tutorial focuses on the simple case of a fixed resolution for the image pipeline and treats both input and outputs as frame buffers. More advanced stuff like Linux DRM for configurable resolutions and V4L based capturing functionalities will be investigated in future tutorials. 


# System Architecture

The basic architecture of the system is illustrated in the following figure:

![](/assets/contents/embedded/zynqfb_pic2.jpg)


In this design, the DMA directly writes the images from  the“ Video input Frontend”  onto the DDR3 RAM. On the other hand, the images that are to be displayed on the monitor are read from the RAM by the DMA and sent to the “Video output Frontend” to be converted to monitor signals. In this architecture the DMA is the master of transaction and the CPU just configures parameters such as pointers to the framebuffers in the RAM and the image resolutions. 

This basic data flow in the system is the same when we use Linux as the software for the CPU. However, we should tell the Linux operating system about the physical addresses of the framebuffers so that it will not try to allocate them to other things. Furthermore, we need to tell the Linux kernel to connect a simple framebuffer driver to these assigned addresses so that we can read from them or write to them later in the software. 

But how we tell linux this things? The answer is through device trees. As I explained in the device trees tutorial, linux finds out about the hardware through reading the device tree file where the structure of the system is defined. 


## Configuring the Device Tree

For this tutorial we’re going to use petalinux as the tool for creating our custom linux system. Thus, if the reader is not familiar with this tool, I suggest reading the petalinux tutorial first before going any further. 

When we create a petalinux project and give it the hardware definition file from our Vivado hardware design, the toolchain automatically creates a device tree describing that design. In addition to that, the system creates a specific .**dtsi** file where we can add our own modifications to the devicetree. This file is called **system-user.dtsi **and is located in the following path:


```
<petalinux project root>/project-spec/meta-user/recipes-bsp/device-tree/files/
```


First, in order to define the reserved memory for the framebuffers, we need to add a **reserved-memory **node to the root of the devicetree. In this node we define two other child nodes representing the addresses and the ranges for the two framebuffers in the system:


```
reserved-memory {
#address-cells = <1>;
#size-cells = <1>;
ranges;
    	hdmi_fb_reserved_region@1FC00000 {
        	compatible = "removed-dma-pool";
        	no-map;
        	// 512M (M modules)
        	reg = <0x1FC00000 0x400000>;
        	// 128M (R modules)
        	//reg = <0x7C00000 0x400000>;
    	};
    	camera_fb_reserved_region@1F800000 {
        	compatible = "removed-dma-pool";
        	no-map;
        	// 512M (M modules)
        	reg = <0x1F800000 0x400000>;
        	// 128M (R modules)
        	//reg = <0x7800000 0x400000>;
    	};
};
```


This defines two framebuffers in the memory. First is the **hdmi_fb_reserved_region** which is for the video output framebuffer and **camera_fb_reserved_region** which is the framebuffer for the video input.

Next, we need to define two device nodes for the **simple-framebuffer** driver which will appear as **fb** devices in the **/dev** directory:


```
hdmi_fb: framebuffer@0x1FC00000 {       	// HDMI out
    	compatible = "simple-framebuffer";
    	reg = <0x1FC00000 (1280 * 720 * 4)>;	// 720p
    	width = <1280>;                     	// 720p
    	height = <720>;                     	// 720p
    	stride = <(1280 * 4)>;              	// 720p
    	format = "a8b8g8r8";
    	status = "okay";
};

camera_fb: framebuffer@0x1F800000 {     	// CAMERA in
    	compatible = "simple-framebuffer";
    	reg = <0x1F800000 (1280 * 720 * 4)>;	// 720p
    	width = <1280>;                     	// 720p
    	height = <720>;                     	// 720p
    	stride = <(1280 * 4)>;              	// 720p
    	format = "a8b8g8r8";
};
```


Note that the **reg** property of these two devices point to the memory ranges that we reserved for the framebuffers in the previous step.

So the Final system-user.dtsi is as follows:


```
/include/ "system-conf.dtsi"
/ {
};

/ {
	#address-cells = <1>;
	#size-cells = <1>;
	reserved-memory {
    	#address-cells = <1>;
    	#size-cells = <1>;
    	ranges;
    	      hdmi_fb_reserved_region@1FC00000 {
        	compatible = "removed-dma-pool";
        	no-map;
        	reg = <0x1FC00000 0x400000>;
    	      };
    	      camera_fb_reserved_region@1F800000 {
        	compatible = "removed-dma-pool";
        	no-map;
        	reg = <0x1F800000 0x400000>;
    	      };
	};





	hdmi_fb: framebuffer@0x1FC00000 {       	// HDMI out
    	compatible = "simple-framebuffer";
    	reg = <0x1FC00000 (1280 * 720 * 4)>;	// 720p
    	width = <1280>;                     	// 720p
    	height = <720>;                     	// 720p
    	stride = <(1280 * 4)>;              	// 720p
    	format = "a8b8g8r8";
    	status = "okay";
	};

	camera_fb: framebuffer@0x1F800000 {     	// CAMERA in
    	compatible = "simple-framebuffer";
    	reg = <0x1F800000 (1280 * 720 * 4)>;	// 720p
    	width = <1280>;                     	// 720p
    	height = <720>;                     	// 720p
    	stride = <(1280 * 4)>;              	// 720p
    	format = "a8b8g8r8";
	};

};
```


Next, we need to enable the **CONFIG_FB_SIMPLE**  modules in the kernel. For this we use the “**petalinux-config -c kernel” **command which brings up the kernel menuconfig for us. By default, the kern will use this our newly created framebuffer as the console for printing kernel messages. We can easily deactivate this functionality by deactivating the **FRAMEBUFFER_CONSOLE **module in the kernel configuration. 


## Configuring the VDMA

The final step toward getting the video IO working is configuring and enabling the VDMA block. We can do this in two ways. It can either be done in the FSBL and before the system boots or using the Xilinx VDMA drivers and through a user space application. 


### Activation through FSBL

The Xilinx FSBL runs a file called **fsbl_hooks.c **which can contain the user codes for doing arbitrary initial configurations to the system. The API for doing this in the FSBL is based on the standalone xilinx drivers and is the same as the code we wrote for configuring the DMA in the previous tutorial. This custom FSBL should be created within the SDK environment. Finally, we package the linux system using the **petalinux-package **command. But, instead of using the automatically generated** zynq_fsbl.elf**, we use our custom fsbl from the SDK. 


### Activation through VDMA drivers

To be done …


# Testing the system

Finally, we boot the system and every thing has been done correctly, we should see our monitor turning on and displaying the linux prompt. We can easily write to and read from the framebuffer in the terminal using the cat command:


```
cat /dev/random > /dev/fb0   #fill the buffer with random numbers
cat /dev/fb1 > /dev/fb0 	#copy the frame from video input to the display
```


One final note is that we could tie the video input and outputs directly together by simply giving the read and write channels of the VDMA a single framebuffer address.
