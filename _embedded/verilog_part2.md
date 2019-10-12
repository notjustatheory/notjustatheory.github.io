---
title: "Verilog Quick and Dirty (Part2: Design philosophy)"
excerpt: "A Quick Guide to Verilog HLD Language"
header:
  teaser: /assets/contents/embedded/verilog_logo.png 
---


## Introduction:

When we design a digital circuit, we can think about the project in two different levels. We can consider all the gates and intricate connections between them or we can think about our design in terms of the counters, comparators, adders  and other submodules. In developing digital designs, the later is called RTL which stands for “register transfer level”. When we use the Verilog for defining our circuit we often use the RTL level. However, as it will be presented in the HLS (High level synthesis) series, nowadays people do not even have to stay in this level and they can design their hardware accelerators and coprocessors in C++ language and just focus on the high level functionality of their design. However, there are some particular hardware projects where managing tight timings of the signals and their low level interactions is of great importance. For example A new CPU architecture or a fast IO processor module are such cases. In this cases, tools like HLS do not provide enough access to the hardware level design and that’s why Verilog and other similar HDL languages maintain their importance. In this series, I’m going to give you a quick guide on the RTL level thinking and present you with some coding techniques that will make the life easier when describing the project using Verilog.


## Data Flow and Controller

A typical digital hardware design comprises of two main subsections. The data flow and controllers. This is somehow a systematic and architectural point of view to the project. We know in almost every digital hardware there is some kind of data processing and information exchanging. For example, consider an MCU. In a microcontroller there is the memory units in which the opcodes and data are stored, there is the ALU (Arithmetic and Logic Unit) that performs the processings and there is a handful of peripheral IO’ s that enables us to interact with the outside world. Now, all this data from these modules flow to each other through some kind of BUS structure. On the other hand there are some management units that tell the different parts of the system when to talk or listen to information on there buses. This units are called controllers. This way of thinking is exactly extended when we go deeper into lower level submodules. For example a PWM generator module might have a register for storing the value of duty cycle and a counter that count up with a clock. On the other hand there is a controller logic that compares the value of the counter and the register and if they math it resets the counter and toggles a latch which is mapped outside the chip as the PWM output. So , When we think about a big project, we actually are breaking it down to units as such and when it comes to defining it using the HDL, we start from bottom and work our way out to other high level units and finally we connect them all together to realize our final project. 


## Designing the controller

Controllers are like law books that monitor a bunch of inputs and internal states and create corresponding outputs based on some kind of logical law. Thus, when it comes to designing them, the thinking process is very much like thinking about a Flowchart rather than a hardware. As a result of this fact, controllers are often defined behaviourally using state machines and ASM charts. Finally,  when coding our controller, we implement these state machines using procedural statements such as **case **and **if-else**. In this blog, I want to give a quick and simple approach toward designing controllers using ASM charts.


### ASM Charts

ASM charts are very much like a flow chart. It has three main building blocks:

**1- State Boxes**

each state box represent an state of the system and is presented through a box with a name or number in it:

![](/assets/contents/embedded/verilog_pic2.jpg)


The flashes indicate that we can transient form one state to another.

**2- conditions**

Using conditions we can check for different inputs and variables in the system and route the transients between states accordingly. 

![](/assets/contents/embedded/verilog_pic3.jpg)


**3- Operation ellipses**

Operations are on the transient paths and do things like assigning outputs, storing a variable or any other thing. Note that in a single operation block, many operations can take place.

![alt_text](/assets/contents/embedded/verilog_pic4.jpg)


** **Now, Combining this tree elements we can define any flowchart and algorithm that we want. In the end, when the flowchart is completed, we have N state boxes. In the code implementation of the Chart, each state will correspond to a particular state in the case statement (so many states in one sentence :-)).


### Example:

Imagine we want to design a circuit that reads a push button and whenever it’s pressed, it lights up an LED for a certain amount of time. For time keeping imagine that we have designed a timer whose timeout can be set and has a preset input and timeout output. The ASM chart for this design would be like this:

![alt_text](/assets/contents/embedded/verilog_pic5.jpg)


As we can see, the chart is made up of two states thus, a 1-bit state register would be enough for the implementation. The verilog code for this state machine is as follows:


```
always @(posedge clk) 
begin 
case(state)
0: 
begin 
	if (key == 1'b1) 
begin
		led=1'b1;
		set_timer() // Pseudo code
		state=1'b1;
	else 
		state=1'b0;
end 
1:
begin 
	if (timeOut==1'b1)
	begin 
		led=0;
		state=0;
	end
	else 
		state=1'b1;
end
default:
	state=0;
endcase
end 
```


One last important thing to point out is that since we are dealing with a hardware here, the **else **in the **if **statements and the **default **in the **case **statement is of paramount importance. The reason is that, these statements grantee that from whatever state we start our circuite, the state machine will fall into the correct internal states. This is because unlike a software code, for every condition some kind of hardware is going to be produced and as a result, a missing else of default would be implemented like an ambiguously connected hardware which can stuck due to some  kind of noise or bad initial conditions that pushes it into undefined scenarios. 


# Creating and Simulating in ISE

In this section we want to create a simple project in the ISE software. The block we want to create is a simple counter that counts the clock signals if its enable input is high. The Verilog code for this block is as follows:


```
module counter(
	input clk,
	input rst,
     input enable,
	output [7:0]cnt
	);
     reg [7:0]count_val;
     assign cnt = count_val;  
    always @(posedge clk)
    begin
   	 if(rst==1'b1)
   		 count_val=0;
   	 else
   		 count_val=count_val+enable;
    end
endmodule
```


First, we create a new project in the ISE and select an appropriate FPGA as the part we want to work with:

![](/assets/contents/embedded/verilog_pic6.jpg)


Next, in the Hierarchy toolbox on the left of the screen we right click on the **xc6slx9-2ftg256 **and select the “New Source” option. 

Now we select the Verilog module item and allocate an appropriate name to it:

![](/assets/contents/embedded/verilog_pic7.jpg)


Now we should copy in our Verilog code. If everything has been done correctly, running the synthesize command will lead to success:

![](/assets/contents/embedded/verilog_pic8.jpg)


Now that the design has synthesized successfully, we go to the simulation section:

![](/assets/contents/embedded/verilog_pic9.jpg)


Similar to the previous section, we create a new file. But, this time we should select the Verilog test fixture option instead. In this dialog window that pops up select the module that we want to simulate which in  our simple example it is the counter module. Next, we modify the testbench to the following code:


```
`timescale 1ns / 1ps
module Testbench;

    // Inputs
    reg clk;
    reg rst;
    reg enable;

    // Outputs
    wire [7:0] cnt;

    // Instantiate the Unit Under Test (UUT)
    counter uut (
   	 .clk(clk),
   	 .rst(rst),
   	 .enable(enable),
   	 .cnt(cnt)
    );
    always #1clk=~clk;
    
    initial begin
   	 // Initialize Inputs
   	 clk = 0;
   	 rst = 1;
   	 enable = 0;

   	 // Wait 100 ns for global reset to finish
   	 #10;
   	 rst=0;
   	 enable=1;
   	 #100;
   	 // Add stimulus here

    end
 	 
endmodule
```


Finally, we select the test bench file and run the “simulate the behavioural model” tool:

![](/assets/contents/embedded/verilog_pic10.jpg)


As a result of running this tool, the logic analyzer and simulator software will pop up as follows:

![](/assets/contents/embedded/verilog_pic11.jpg)


We can see that our model works as expected in the simulation. 


## Assigning Pins in ISE

When the design is done and successfully simulated, We deploy it to the hardware. But before that, we need to assign the pins we want to use. To do this, we use a constraint file with **.ucf** format one of whose usages is assigning pins. The syntax is as follows:


```
NET "Pin name in our design" LOC = "Pin name on the FPGA";
NET "clk" LOC = "P91";
NET "LED" LOC = "P17";
```



# Creating and Simulating in Vivado

Coming soon ...
