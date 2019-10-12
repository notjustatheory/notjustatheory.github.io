---
title: "Verilog Quick and Dirty (Part1)"
excerpt: "A Quick Guide to Verilog HLD Language"
header:
  teaser: /assets/contents/embedded/verilog_logo.png
---

# Verilog Basic:

This is not intended to be a comprehensive guide on Verilog, but this is a quick and dirty guide on how to start coding and building real and cool projects. The stuff covered in this guide begin with the basic syntax and conclude with some simple examples. 


## Basic structure:

When defining a hardware, what we actually are thinking about is a block (digital circuit) with a couple of inputs and outputs that perform some kind of operation. For example, adding two numbers or performing some sequential decision making logic. 

![](/assets/contents/embedded/verilog_pic1.png")


Interestingly, this block itself could be comprised of many other smaller blocks for doing simpler tasks. Thus, it seems like this blocks should be the basic structures of the hardware description language, like sentences in a novel! In fact it is actually how things are with Verilog too. For defining a block in Verilog we use modules. 


### Module :

We define our block like this:


```
module magic_module(in1, in2, out1);
...
endmodule 
```


The first thing we need to define at the beginning of the module is the properties of each input and output. After all, we need to tell the synthesizer ( A fancy name for the software that makes the hardware out of the code) which argument is an input or output  and how wide they are ( how many bits there is in a single output or input ). Let’s imagine we want our module to be a simple adder that adds the two inputs and outputs the result. Here we set the inputs to be 4 bits and the output to be 5 ( when adding two 4 bit numbers, the output is at most 5 bits). So let’s complete the basic module like this:


```
module magic_module(in1, in2, out1);
input [4:0]in1 , [4:0]in2;
output  [5:0]out1;
...
endmodule 
```


Before completing our module lets learn some other things first.


## Numbers in Verilog


### Representing numbers

On many occasions we need to do operations and comparisons where numbers are involved, just like any other programming language. So, here the syntax for writing down numbers is shown:


```
2'b1z //a two-bit binary number
2'o17 // a two digit ( 6 bits) number
3'h2dc // a tree digit (12 bits) number
9 or 'd9 // a normal decimal number
```



### Wires and Registers

As we know, in a digital circuite numbers are presented using signals that are either 0 ( 0 volts) or 1 ( 3.3 V or other standard voltages). So, in a digital circuit, a bunch of wire can be actually a number! But wait, couldn’t a wire be just floating and connected to nowhere? What do we call it? In fact, we do have such a thing as floating and this is represented a **‘z’ **in the number. For example **3’b01z **is a symbol for a number whose first wire is not derived and is floateing. In the simple module example, inputs and outputs are actually representing wires in the circuit. But those are that magic wires that come out of our module. What if we want to define some set of wires to use inside our module? wires that do not come out. To do this, we use the **wire **keyword like so:


```
wire [32:0]data;
```


Here, data is the representation of a fistful of 32 wires each corresponding to one binary digit in the “data”. 

On the other hand we have registers. Unlike wires, registers are like memories that maintain their state. They can change on the occurrence of special events in our circuit and using them we can make circuits that are moody and know about their stat! So with them, we can build pretty cool sequential circuits. Registers are defined using **reg** keyword:


```
reg [32:0]my_register; // A 32 bit register called my_register 
```



## Operators in Verilog:


### Arithmetic operations:

For doing simple math operations there are builtin agents in the verilog. These are:


```
c=a*b //multiply
c=a/b //int divide by
c=a+b //simple addition
c=a-b //subtract
c=a%b // a mod(b)
```



### Bitwise Operations:


```
& //and
| //or
^ //xor
~&  //nand
~|  //nor
~^  //xnor
```



### Relational operators

This operators return a 1 bit result which is 1 if true and 0 if false. Just like a boolean: 


```
< //is less than
> //is more then
>=  //less than or equal
>=  //more than or equal
&&  // is both true?
||  //is any one true?
! //return the inverses, make true if false and visevera 
```



### Shift and other important operators


```
<<  // a << 1, shift left 
>>  //shift right
{}  //concatenate two numbers and create one whose width is the sum of the other two
    //Example: {co, sum} = a+b+ci
{ {} }  //replicate the thing in the inner as many as the number in the  outer
    //Example b = {3{a}}
?:  // c=sel ? a : b; // if sel is true c=a else c=b. This is called ternary operator
```



## Assignments

In verilog there are two kinds of assignments for values, procedural and continuous .


### Continuous Assignment

This kind of assigning values implements as a combinational circuit. It means that as soon as one side is changed, the change is applied to the other side. This is actually like connecting wires to each other and it  is done like this:


```
assign mynet = value 
```



#### Example: 

Let’s hold on a minute and complete or simple adder using the things we’ve learned so far. We wanted our adder to output the result as soon as its inputs change. Guess what, this is a combinational circuit with continuous assignment:


```
module magic_module(in1, in2, out1);
input [4:0]in1 , [4:0]in2;
output  [5:0]out1;
assign  out1 = in1+in2;
endmodule 
```



### Procedural assignment

The other important type of assignment is the “procedural assignment”. When we were talking about registers I told that they can change only  on the occurrence of special events. Thus, when that event occurs, the register sets to a particular value. This kind of assignment is called a procedural assignment. The most useful way of implementing this is using the “**always**” block. This block is employed like this:


```
always @(posedge clk or reset)
begin 
...
end 
```


Here, the list of events and signals in the parentheses after the **@** is the sensitivity list. It means that whenever any of these sensitivity conditions happen, the code inside the block runs. In this example, the sensitivity list is the positive edge of the clock signal “or” any change of the reset signal. Now, inside the block we can easily do things like conditional statements and register assignments. For example, Imagine the following code:


```
module adder(a, b, ci, co, sum, clk);
Input a, b, ci, clk;
output co, sum;
reg co, sum;
always @(posedge clk)
  {co,sum} = a+b+ci;
endmodule 
```


This example is a simple synchronous 1 bit full adder. On the assertion of clk, the code inside the always block runs which in this case is a simple arithmetic operation. 

Now here it comes the confusing terms of blocking and non-blocking assignments. In the example above, the way we assigned the variables **co **and **sim** is called blocking assignment. There is also another type of assignment called non-blocking assignment which is different. Let’s first demystify this two terms.


#### **Blocking assignments:**

The **“=”** token presents a blocking assignment. When we use this type of assignment, the execution flow within the procedure is blocked until the assignment is completed. Similarly, the evaluation of the concurrent statements are also blocked until the assignment is completed. What this simply means is that, the process of the execution of the code when we use this type of assignment is pretty much like when we’re programming in a normal programming language like C, codes run one after another. 


#### **Non-blocking assignment:**

On the other hand, we have the non-blocking assignments. Non-blocking assignment is presented using the **“<=” **token. When we use this, the assignment happens in two steps; First, the value on the right-hand side of all the assignments in the procedural section (the **always’s  **codeblock) is evaluated immediately. Next, the assignment to the left-hand side is postponed until other evaluations in the current time step are completed. 

**Example:**

The following code doesn't work.


```
always @(posedge clk)
begin 
word[15:8] = word[7:0];
word[7:0] = word[15:8];
end  
```


But, the code below uses the non-blocking assignment thus it works.


```
always @(posedge clk)
begin 
word[15:8] <= word[7:0];
word[7:0] <= word[15:8];
end 
```


The reason is simple. In the first example, first the evaluation and assignment is performed of the first half of the word and when it comes to the second line of code, the value of the word[15:8] is lost. On the other hand, when we use the non-blocking statement, the right-hand-side of the assignments are evaluated first and next, in one step, the values are assigned to the left-hand-sides. This way, no information is lost.

Let’s see another example:


```
always @(posedge clk)
A=A+1;
always @(posedge clk)
B=A+1;
```


This code yields unpredictable results. That’s because the new value of B could be evaluated before or after the A is changed. On the contrary, the following code will produce predictable results:


```
always @(posedge clk)
A<=A+1;
always @(posedge clk)
B<=A+1;
```


Here, the new value of B will always be evaluated before A changes. 


### Defining combinational circuits using procedural statements:

So far, circuites defined from the procedural code was of sequential digital circuit types. Now what happens if we make the always block sensitive to all the signals whose change, changes the result. For example consider the following code:


```
module adder(a, b, ci, co, sum);
Input a, b, ci;
output co, sum;
reg co, sum;
always @(a , b , ci)
  {co,sum} = a+b+ci;
endmodule 
```


Yeay, what we’re defining here is simply  a combinational full adder!


## 


## Conditional statements

The two most frequently used conditional statements are **if-else **and **case** statements.


### If-else:

The syntax for this statement is as follows:


```
if (condition) begin
...
end 
else if(statement) begin
...
end else begin 
... 
end
```



### Case statement:


```
case(select)
case  [case-expr}
[item]  :
begin
[procedural statement];
[procedural statement];
. .  .
end
[item]  :
begin
[procedural statement];
[procedural statement];
.  . .
end
[iteml]  :
begin
[procedural statement];
[procedural statement];
end
. . .
default:
begin
[procedural statement];
[procedural statement];
end
endcase
```


It is worth knowing that there exists two other case statements namely **casex **and **casez**. The first, treats ‘z’ and ‘x’ values as “don’t care” and the next does the same thing just for the ‘x’ values.


## Instantiating a modules:

As I said, when we think about a digital design, we somehow break it into smaller and simpler building blocks that in Verilog are called modules. Being like Legos, when we develop a project, we first design the modules, and next we put them together in a bigger higher level module which is actually our final digital circuit. This final module is often called the **“Top module”**. Here, I want to show how we can instantiate a previously defined module within another one. For example consider a module “**child_module” ** that is to be instantiated in the **“parent_module”. **For this we write:


```
module child_module( input clk, input rst_n, input [9:0] data_rx, output [9:0] data_tx );

module parent_module( input clk, input rst_n, input enable, input [9:0] data_rx_1, input [9:0] data_rx_2, output [9:0] data_tx_2 );
parent_module child_module_instance_name ( clk, rst_n, data_rx_1, data_tx ); 
endmodule
```


In this example, the child_module_instance_name is the instantiated module. However, if for some reason the order of the arguments in the definition of the child_module is changed, the connections will break. Thus, a more robust way of instantiating the child module would be like this:


```
parent_module child_module_instance_name ( .clk(clk), .rst_n(rst_n), .data_rx(data_rx_1), .data_tx(data_tx) );
```



## This is enough for most of the projects!

So far we have learned enough Verilog syntax to address our needs in a world of projects. However, Verilog is a full fledged programming language with books with hundreds of pages explaining it. Here, some useful links and resources is presented:



*   [The reference designer website](http://referencedesigner.com)
*   [FPGA For Fun website](https://www.fpga4fun.com/)
*   [Stanford’s Verilog Cheat Sheet](https://web.stanford.edu/class/ee183/handouts_win2003/VerilogQuickRef.pdf)

Next in this blog series, I’ll explain some useful coding and designing techniques which exploit the syntaxes we’ve learned so far for designing cool circuits. 