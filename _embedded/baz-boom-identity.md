---
title: "A Quick Hands-on Device trees for Embedded Linux applications"
excerpt: "For defining the hardware structure of an embedded system, We must know about device trees. In this blog, I document what I learn about this."
header:
  teaser: assets/images/unsplash-gallery-image-1-th.jpg
---
# Introduction 

The power of linux , as in an embedded linux context, comes from the huge pool of open source codes that are freely available to the developer. When we say embedded linux we are often thinking about a tiny computer which is  connected to a bunch of sensors, actuators and other kinds of hardware. Having their own mechanisms and operation principles, each of these devices require a driver in order to operate properly. In a conventional micro controller based system, all of these drivers have to be hand crafted and ported by the developer which introduced a huge load to the development process. However, Linux provides a global standard that makes things much easier. In Linux, the drivers act as an abstractor layer between the user of the device. In fact in Linux, all the device are presented to the user as files through which user can interact with the device and exchange data with it. 

Linux device drivers for thousands of devices are freely available on the net. So when we develop an embedded linux system, we seldom need to write any drivers and we can easily use the existing  ones. However, in order to do this, we need to know how to tell Linux which hardwares are connected to it and in what ways. In other words, as the linux boots, it needs to know what devices there are  so that it knows what drivers to invoke. 

The task of defining the hardware structure of the system is done by file called device tree bob. The bootloader of an embedded linux system ( for example U-BOOT) passes this file to the kernel as it boots and through this linux knows what types of hardware and system structure it’s dealing with. This binary .dtb file is compiled from .dts source in which the structure of the system is defined in tree like data structure with a special syntax. 

Thus, the process of adding a device to the embedded linux system is as follows:



1. If the driver is not included in the kernel as a module or built in, compile the kernel with the required driver enabled.
2. Modify the default device tree source file of the particular board or SOC and add the new device to it.
3. Compile the device tree using the device tree compiler (dtc) and create the device tree binary file (dtb)
4. Replace the old dtb with the new one and linux should now recognize the new hardware as it boots

In this document, I intend to write a short tutorial on how syntax is and how we can add a new driver to our system. 
# The syntax and basic structures

In one picture, the basic syntax can be illustrated as follows[]:

![Basic Sturcture of a Device Tree](/assets/contents/embedded/dt_basic_structure.png)
The system is defined by a hierarchy of nodes with the root node ( **/** ) as the parent of the whole structure.  The contents for each hierarchy is defined between a block indicated by braces ( **{} **). 

Each node in the system has a name and address defined as **name@address**. The address points to the physical location of the device in the memory map of the SOC we’re working with. Generally, each node in the system can have multiple properties that define the type of the device ( the node ) and its connections to the system. Moreover, each node in the system can have a label through which we can refer to it later. This label is defined as a name followed by a colon that are placed before the node’s name. Later, we can refer to the node using their labels followed after an ampersand (&lable).


## Overwriting a property 

if we want to overwrite a property of a node, we can easily refer to it with its label and overwrite its properties as we wish. This is illustrated in the following code where the dr_mode of the usbdev0 is overwritten:


```
&usbdev0 {
    dr_mode = "host";
};
```



## Activating/Deactivating devices

An important property of a device is its activation status. This parameter can be set as **“okey” ** or **“disabled”**. This is illustrated as follows:


```
&uart4 {
    status = "okay";
};
```



## Overwriting nodes

Nodes can be overwritten just as properties by redefining them in a later stage. For example, in the following the **vf610-colibri **is overwritten:


```
&iomuxc {
    vf610-colibri {
        pinctrl_uart2: uart2grp {
            fsl,pins = <
                VF610_PAD_PTD0__UART2_TX        0x21a2
                VF610_PAD_PTD1__UART2_RX        0x21a1
            >;
        };
...
    };
};
```



## Deleting properties and nodes

It is also possible to delete properties or even nodes using **_/delete-property/_ **or **_/delete-node/_**. The following example deletes the **fsl,uart-has-rtscts** property defined in the carrier board level device tree **imx6qdl-colibri.dtsi**:


```
&uart1 {
    /delete-property/fsl,uart-has-rtscts;
};
```



```
/delete-node/backlight;
```



## Aliases

In the **aliases** section of a DTS, we see entries of the format:


```
property = &label;
```


It basically assigns the value of **label** to **property**. Henceforth, the long-name to the node identified by the label can be accessed using the shorthand property. The RHS of this assignment is using labels and **NOT** the short-names of the individual nodes. A label in DTS refers to the individual node (using its complete long path). 

For example in the following, the property **ethernet0** is now set to **"/soc8313@e0000000/ethernet@24000" **which is the long path for the **&enet0:**


```
/dts-v1/;
/ {
    model = "MPC8313ERDB";
    compatible = "MPC8313ERDB", "MPC831xRDB", "MPC83xxRDB";
    #address-cells = <1>;
    #size-cells = <1>;
    aliases {
        ethernet0 = &enet0;
        serial0 = &serial0;
        pci0 = &pci0;
    };
```



## Compatible property

Every node in the tree that represents a device is required to have the **compatible** property. The operating system uses the compatible property to decide which driver to bind to the device. Compatible is a list of strings. The first string in the list specifies the exact device that the node represents in the form **"<manufacturer>,<model>"**. The following strings represent other devices that the device is _compatible_ with. For example the serial node in the following code is compatible with **pl011 **device:


```
   serial@101f0000 {
        compatible = "arm,pl011";
        reg = <0x101f0000 0x1000 >;
    };
```



## The Addressing Mechanism

Devices that are addressable in the system, use the following properties to encode address information:



*   reg
*   #address-cells
*   #size-cells

Each addressable device gets a reg which is a list of tuples in the following form 


```
reg = <address1 length1 [address2 length2] [address3 length3] ... >;
```


Each tuple represents an address range used by the device. Each address value is a list of one or more 32 bit integers called cells. Since both the address and length fields can have variable cell sizes, the **#address-cells** and **#size-cells** properties in the parent node are used to state how many cells are in each field. This type of addressing is performed for memory mapped devices. These devices respond to the range of addresses defined by the reg property. 



This is illustrated in the following example:


```
/ {
    #address-cells = <1>;
    #size-cells = <1>;

    ...

    serial@101f0000 {
        compatible = "arm,pl011";
        reg = <0x101f0000 0x1000 >;
    };
};
```


Another simple example for a memory mapped device is the CPU:


```
   cpus {
        #address-cells = <1>;
        #size-cells = <0>;
        cpu@0 {
            compatible = "arm,cortex-a9";
            reg = <0>;
        };
        cpu@1 {
            compatible = "arm,cortex-a9";
            reg = <1>;
        };
    };
```


In the cpus node, **#address-cells** is set to 1, and **#size-cells** is set to 0. This means that child reg values are single uint32 numbers that represent the address with no size fields. In this case, the two cpus are assigned addresses 0 and 1.

It is notable that the reg values matches the value in their corresponding  node names. This is because by convention, if a node has a reg property, then the node name must include the unit address, which is the first address value in the reg property. 


## 


## Non Memory Mapped Devices

Non memory mapped devices can have address ranges, but they are not directly accessible by the CPU. Instead the driver for the parent of the non memory mapped device, performs indirect access to the device on behalf of the CPU. 

I2C devices are examples of this paradigm an example of which is illustrated in the following:


```
       i2c@1,0 {
            compatible = "acme,a1234-i2c-bus";
            #address-cells = <1>;
            #size-cells = <0>;
            reg = <1 0 0x1000>;
            rtc@58 {
                compatible = "maxim,ds1338";
                reg = <58>;
            };
        };
```


Here, the DS1338 I2C device is assigned an address, but there is no length or range associated with it.


### Ranges (Address Translation)

We've talked about how to assign addresses to devices, but at this point those addresses are only local to the device node. It doesn't yet describe how to map from those address to an address that the CPU can use. 

The root node always describes the CPU's view of the address space. Child nodes of the root are already using the CPU's address domain, and so do not need any explicit mapping. For example, the **serial@101f0000** device is directly assigned the address **0x101f0000**. On the other hand, nodes that are not direct children of the root do not use the CPU's address domain.** **

In order to get a memory mapped address the device tree must specify how to translate addresses from one domain to another. The ranges property is used for this purpose.

Here is a sample device tree with range property added:


```
/dts-v1/;

/ {
    compatible = "acme,coyotes-revenge";
    #address-cells = <1>;
    #size-cells = <1>;
    ...
    external-bus {
        #address-cells = <2>
        #size-cells = <1>;
        ranges = <0 0  0x10100000   0x10000     // Chipselect 1, Ethernet
                  1 0  0x10160000   0x10000     // Chipselect 2, i2c controller
                  2 0  0x30000000   0x1000000>; // Chipselect 3, NOR Flash

        ethernet@0,0 {
            compatible = "smc,smc91c111";
            reg = <0 0 0x1000>;
        };

        i2c@1,0 {
            compatible = "acme,a1234-i2c-bus";
            #address-cells = <1>;
            #size-cells = <0>;
            reg = <1 0 0x1000>;
            rtc@58 {
                compatible = "maxim,ds1338";
                reg = <58>;
            };
        };

        flash@2,0 {
            compatible = "samsung,k8f1315ebm", "cfi-flash";
            reg = <2 0 0x4000000>;
        };
    };
};
```


**ranges** is a list of address translations. Each entry in the ranges table is a tuple containing the child address, the parent address, and the size of the region in the child address space. For the external bus in our example,  the three ranges are being translated as follows:



*   **Offset 0** from **chip select 0** is mapped to address range **0x10100000 - 0x1010ffff**
*   **Offset 0** from **chip select 1** is mapped to address range **0x10160000 - 0x1016ffff**
*   **Offset 0** from **chip select 2** is mapped to address range **0x30000000 - 0x30ffffff**

Alternately, if the parent and child address spaces are identical, then a node can instead add an empty ranges property. The presence of an empty ranges property means addresses in the child address space are mapped 1:1 onto the parent address space. 

Note that in this example there is no ranges property in the i2c@1,0 node. The reason for this is that unlike the external buses, devices on the i2c bus are not memory mapped on the CPU's address domain. Instead, the CPU indirectly accesses the rtc@58 device via the i2c@1,0 device. The lack of a ranges property means that a device cannot be directly accessed by any device other than it's parent.


## Interrupts

Unlike address range translation which follows the natural structure of the tree, Interrupt signals can originate from, and terminate on any device in a machine. Unlike device addressing which is naturally expressed in the device tree, interrupt signals are expressed as links between nodes independent of the tree. 

Four properties are used to describe interrupt connections: 



*   **interrupt-controller** 

    An empty property declaring a node as a device that receives interrupt signals

*   **#interrupt-cells** 

     A property of the interrupt controller node.which states how many cells there are in an _interrupt specifier_ for this interrupt controller.

*   **interrupt-parent** 

    A property of a device node containing a **_phandle_** to the interrupt controller that it is attached to. Nodes that do not have an **interrupt-parent** property can also inherit the property **from their parent node**.

*   **interrupts** 

    A property of a device node containing a list of _interrupt specifiers_, one for each interrupt output signal on the device.


The meaning of an interrupt specifier depends entirely on the binding for the interrupt controller device. Each interrupt controller can decide how many cells it need to uniquely define an interrupt input.

The following example adds the interrupt connections to the example from previous section:


```
/dts-v1/;

/ {
    compatible = "acme,coyotes-revenge";
    #address-cells = <1>;
    #size-cells = <1>;
    interrupt-parent = <&intc>;

    cpus {
        #address-cells = <1>;
        #size-cells = <0>;
        cpu@0 {
            compatible = "arm,cortex-a9";
            reg = <0>;
        };
        cpu@1 {
            compatible = "arm,cortex-a9";
            reg = <1>;
        };
    };

    serial@101f0000 {
        compatible = "arm,pl011";
        reg = <0x101f0000 0x1000 >;
        interrupts = < 1 0 >;
    };

    serial@101f2000 {
        compatible = "arm,pl011";
        reg = <0x101f2000 0x1000 >;
        interrupts = < 2 0 >;
    };

    gpio@101f3000 {
        compatible = "arm,pl061";
        reg = <0x101f3000 0x1000
               0x101f4000 0x0010>;
        interrupts = < 3 0 >;
    };

    intc: interrupt-controller@10140000 {
        compatible = "arm,pl190";
        reg = <0x10140000 0x1000 >;
        interrupt-controller;
        #interrupt-cells = <2>;
    };

    spi@10115000 {
        compatible = "arm,pl022";
        reg = <0x10115000 0x1000 >;
        interrupts = < 4 0 >;
    };

    external-bus {
        #address-cells = <2>
        #size-cells = <1>;
        ranges = <0 0  0x10100000   0x10000     // Chipselect 1, Ethernet
                  1 0  0x10160000   0x10000     // Chipselect 2, i2c controller
                  2 0  0x30000000   0x1000000>; // Chipselect 3, NOR Flash

        


        ethernet@0,0 {
            compatible = "smc,smc91c111";
            reg = <0 0 0x1000>;
            interrupts = < 5 2 >;
        };

        i2c@1,0 {
            compatible = "acme,a1234-i2c-bus";
            #address-cells = <1>;
            #size-cells = <0>;
            reg = <1 0 0x1000>;
            interrupts = < 6 2 >;
            rtc@58 {
                compatible = "maxim,ds1338";
                reg = <58>;
                interrupts = < 7 3 >;
            };
        };

        flash@2,0 {
            compatible = "samsung,k8f1315ebm", "cfi-flash";
            reg = <2 0 0x4000000>;
        };
    };
};
```


Some points regarding the example:



*   The machine has a single interrupt controller at 10140000.
*   The label **'intc:'** has been added to the interrupt controller node, and the label is used to assign a phandle to the **interrupt-parent** property in the root node. This interrupt-parent value becomes the default for the entire system because all child nodes inherit it unless it is explicitly overridden.
*   Each device uses an **interrupt** property to specify a different interrupt input line.
*   **#interrupt-cells** is 2, so each interrupt specifier has 2 cells. This example uses the common pattern of using the first cell to encode the interrupt line number, and the second cell to encode flags such as active high vs. active low, or edge vs. level sensitive. For any given interrupt controller, refer to the controller's binding documentation to learn how the specifier is encoded.


## Device Tree Inclusion

Device Tree files are not monolithic, they can be split in several files, including each other. However, the inclusion paradigm is not similar to the C inclusion. But, here when a file is included, it augments the files to which it is has been included. A graphical representation of this process is illustrated in the following figure[] :



<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/A-Quick3.png). Store image on your image server and adjust path/filename if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](/assets/contents/embedded/dt_inclusion.png)



# Sumerry

In this document I explained the importance of learning about device trees and the possibilities that they for exploiting the huge reservoir of available drivers in the community. Next, I provided a short tutorial regarding the structure and the syntax of device trees that are essential for working with them. This section of the tutorial was heavily taken from references provided at the end of this document. However, the order and the level of details were modified for better explaining  the subject without getting into too much detail. Moreover, I’ve modified them and added my own experiences as well to make the tutorial a little clearer. 

By all means, this post just scratches the surface of the matter and there is a lot to be learned which I believe would only be feasible in the context of doing projects. 