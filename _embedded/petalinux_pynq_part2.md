---
title: "Getting into Petalinux and Pynq (Part1)"
excerpt: "Pynq and PetaLinux for Xilinx SOCs."
header:
  teaser: /assets/contents/embedded/petalinux_logo.png 
---
# Creating A Basic Image Processing Application Using Pynq


## Hardware Design

In this section we want to create a basic project for reading video data from an inexpensive global shutter camera module and receive it in the jupyter environment. The camera sensor we’re going to use is MT9V034 global shutter machine vision image sensor form On Semiconductors. This is shown in the following image:

![](/assets/contents/embedded/petalinux_pic6.jpg )


This sensor is relatively easy to deal with since the pinouts are not of the BGA type and they are easy to be soldered. The output of this camera is sent through parallel video interface. This timing diagram for this sensor is shown in the following image:

![](/assets/contents/embedded/petalinux_pic7.jpg )

![](/assets/contents/embedded/petalinux_pic8.jpg )

As we can see in this diagram, each pixel is put on the parallel 10-bit output of the sensor on the rising edge of the pixel clock signal. Furthermore, each line of the image is transferred during the high value of Line valid and the start of a new frame is indicated by the rising edge of the frame valid signals. So in total we have 13 lines of signal, 10 for Data, 1 for the line valid, 1 for frame valid and one clock signal. For connecting this signals to the FPGA pins we are completely free except for the clock signal. This signal should be connected to a MRCC capable pin.

As we know, in the Vivado design methodology all the signals and data buses are standardized to the AXI interface. As such, the first step of transferring the image into the design would be  converting the parallel video to AXI stream interface. This is shown in the following image:

![](/assets/contents/embedded/petalinux_pic9.jpg )

In the above image, the “Video in to AXI4-Stream” is the block that converts parallel video into AXI stream video data. For adapting the signal from camera to the parallel interface supported by the block we should add some basic logic to convert the Line valid and Frame valid signals into hblank and vblank interfaces. Furthermore, I have designed a simple IP to detect whether the camera is properly connected or not.  This is called cam_line_status in the above picture. This block asserts the line status signal to one in case the video timing signals connected to it are valid and with the right resolution.

In the next step, the AXI video form the “Video In to AXI4-Stream” is applied to the color filter interpolation block. This block is for converting the raw bayer video from the camera to RGB AXI video signal.

So far, the camera data has been prepared to be written to the DDR3 memory. For this we will use a VDMA engine within the design. This Block is responsible reading the pixel data from the AXI video one by one and writing it to the RAM. This block accesses the RAM through AXI HP interface of the PS submodule. 

It is important to note that for the pynq system to be able to work properly with the AXI video form the CFA block,  it has to be modified to the pynq standard memory structure. In the example project, this is done through color_convert and pixel_pack blocks. This is illustrated in the following diagram:

![](/assets/contents/embedded/petalinux_pic10.jpg )

The output “stream_out_32” is the output which has been transferred to the RAM. As I mentioned, this is done using the AXI VDMA module as illustrated in the following figure:


![](/assets/contents/embedded/petalinux_pic11.jpg )
It is important to note that for the Pynq system to work with the VDMA module, its interrupt pins be connected to an AXI interrupt controller. The overall system connections can be seen in the following diagram.


![](/assets/contents/embedded/petalinux_pic12.jpg )

After assigning the camera pins in the IO planner and synthesizing the design, we should export the block diagram and the bitstream files to be used later by the Pynq library.


## Software Design

In this section, we use the hardware files from the last step to create the software.  Pynq, calls this hardware files overlays. Overlays are loaded to the FPGA and as such shapes it into our custom design. In the first step we copy the generated overlay form the last step into an optional location on the pynq sd card. After booting the board and logging into the jupyter notebook, we load the overlay using the following line in the python code:


```
from pynq.lib.video import *
from pynq import PL
from pynq import Overlay
import cv2
import numpy as np
import PIL.Image
import matplotlib.pyplot as plt
base = Overlay("/home/xilinx/Overlay/SOC.bit")
```


The Overlay class loads the bitstream we just created into the FPGA. Later on, we will be able to access our IPs in the design through the “**base” **object.

In the next step we need to initialize the pixel pack and color convert modules:


```
# Pixel Format
pixel_in = base.mt9v034_video_in2.cam_vid_in.pixel_pack
pixel_out = base.mt9v034_video_in2.live_vid_out.pixel_unpack

pixel_in.bits_per_pixel = 24
pixel_out.bits_per_pixel = 24

# Color mapping
colorspace_in = base.mt9v034_video_in2.cam_vid_in.color_convert
colorspace_out = base.mt9v034_video_in2.live_vid_out.color_convert1

bgr2rgb = [0, 0, 1,
           0, 1, 0, 
           1, 0, 0,
           0, 0, 0]

colorspace_in.colorspace = bgr2rgb
colorspace_out.colorspace = bgr2rgb
```


As we can see since the drivers for these two blocks are available in the Pynq package, configuring them is as easy as writing some value into the methods corresponding to them.

In the next phase we should configure the VDMA module. VDMA is also supported by Pynq library and as such we can easily configure it within the pynq system. This is done as follows:


```
cam_vdma = base.mt9v034_video_in2.axi_vdma
framemode = VideoMode(752, 480, 24) # Set the read channel parameters to the camera specifications and enable the VDMA
cam_vdma.readchannel.mode = framemode
cam_vdma.readchannel.start()
```


This will fire the VDMA module which leads the images to be transferred into the RAM. Finally we can plot the images using the python openCV. This is done as follows:


```
frame = cam_vdma.readchannel.readframe()
plt.imshow(frame)
plt.show()
```


The result has been illustrated for the case of a stereo setup in the following image:


![](/assets/contents/embedded/petalinux_pic13.jpg )

As we can see, developing hardware accelerated image processing applications has never been easier before using Pynq and Zynq.
