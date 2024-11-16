- Datatypes : 2- state , 4- state
- Array: Fixed, Queue and Dynamic array
- Array Initialization
- Loops for array
- Queue Method
- Demonstration of Fixed Size array and Queue

![[Pasted image 20241104214746.png]]

![[Pasted image 20241104214806.png]]

```
module tb;

bit a = 0;
byte b = 0; /// 2-state->0 , 4 state -> x
shortint c = 0;
int d = 0;
longint e = 0;

bit [7:0] f = 8'b00000000;
bit [15:0] g = 16'h0000;

real h = 0; 




initial begin


a = 1'b0;

end

```

```
module tb;

byte var1 = -126; /// -128 to +127
bit [7:0] var2 = 130; ////0 to 255

initial begin
#10;
$display("value of VAR : "%0d", var2);
end


shortint var2 = 0;


endmodule
```

```
`timescale 1ns/1ps

module tb;

time fix_time = 0; ///store fixed point time value
realtime real_time = 0; ///store floating point time value

//////
////$time(); ///return current simulation time in fixed format 
//$realtime(); /// return current simulation time in floating point format 

initial begin
#12.23;
fix_time = $time();
$display("Current Simulation Time: "%0t", real_time);
end

////12.34 -> 12
//12.578 -> 13
endmodule
``` 

Implementing Simple 2:1 MUX 
```
`timescale 1ns/1ps

module top(
input a,b,sel //sel = 0, y = a, sel = 1, y = b
output y 
);

reg.temp;

always@(*)
begin
   if(sel == 1'b0)
   temp = a;
   else
   temp = b;
   end
assign y = temp;
   
endmodule
```

Implementing a full adder consisting of two half adders
![[Pasted image 20241105205210.png]]

```
module ha(
input wire a,b;
output wire sout,cout
);

assign sout = a^b;
assign cout = a&b;

endmodule

module fa
(
input a,b,cin,
output s,c
);
wire t1,t2,t3;// replace wire with logic


ha h1(a,b,t1,t2);
ha h2(t1,cin,s,t3);

assign c = t2 | t3;

endmodule
```

```
/*
module top (
input a,b,sel, //sel=0,y=a,sel=1,y=b
output y,
);

logic temp;

always@(*)
begin
if(sel == 1'b0)
temp= a;
else
temp = b;
end
assign y = temp;
endmodule
*/
```

## Understanding usage of an array
![[Pasted image 20241106211008.png]]

```
module tb;

bit arr1[8];
bit arr2[]// = (1,0,1,1);

initial begin
$display("size of arr1: "%0d"l, $size(arr1));
$display("size of arr2: "%0d"l, $size(arr2));

$display("value of first element:"%0d", arr1[0] );
arr1[1]=1;
$display("value of first element:"%0d", arr1[1] );


$display("value of all elements of an arr2:"%0p",arr2);

end

endmodule

```  

## Array Initialization Strategies

![[Pasted image 20241107203752.png]]

```
module tb;
///unique
int arr1[2]='{1,2};
///repetition
int arr2[2] = '{ 2{3} }'
///default
int arr3[2] ='{default : 2};
///uninintialized
int arr4[2];

initial begin
$display("arr: %0p",arr);


end
endmodule
```

## Loops for repetitive array operation

![[Pasted image 20241107205334.png]]

for loop
```
module tb;
int arr[10];///0-9
int i = 0;


initial begin

for(i = 0; i<10; i++) begin
arr[i] = i;
end
$display("arr: %0p",arr);

end
endmodule
```

for each
```
module tb;
int arr[10];///0-9
int i = 0;
initial begin
foreach(arr[j]) begin //0---9
arr[j]=j;
$display("%0d", arr[j]);

end
endmodule

```

repeat
```
module tb;
int arr[10];///0-9
int i = 0;
initial begin

repeat(10) begin
arr[i] = i;
i++;
end
$display("arr:%0p", arr);
end
endmodule
```

## Array Operation : 'COPY'
```
module tb;
 int arr1[5];
 int arr2[5];//Both the arrays must be of the same datatype
 //Storage capacity must be kept in mind (must be exacty same)

initial begin
for(int i = 0, i<5, i++) begin
arr1[i]=5*i; // 0 5 10 15 20
end
arr2 = arr1;
$display("arr2 : %0p",arr2);

endmodule
```

## Array Operation : 'COMPARE'
```
module tb;
 int arr1[5] = '{1,2,3,4,5};
 int arr2[5] = '{1,2,7,4,5};

int status;

initial begin
status = (arr1 != arr2);
$display("Status: %0d", status);
end
 

endmodule
```

## Dynamic Array
```
module tb;

int arr[];
int farr[30];

initial begin
arr = new[5];

for (int i = 0: i < 5; i++)begin
arr[i] = 5*i;
end
//$display("arr : %0p", arr)

arr = new[30] (arr);
$display("arr : %0p", arr)
farr = arr;
$display("farr : %0p", farr)

/* 
arr.delete();
for (int i = 0: i < 5; i++)begin
arr[i] = 5*i;
end
*/
endmodule
```

## Queue
```
module tb;

int arr[$];
initial begin
arr = {1,2,3};
$display("arr : %0p", arr);

arr.push_front(7);
$display("arr : %0p", arr);

arr.push_back(9);
$display("arr: %0p", arr);

arr.insert(2,10);
$display("arr: 0%p", arr);

 j = arr.pop_front();
$display("arr :%0p",arr);
$display("value of j : %0d",j);

j = arr.pop_back();
$display("arr :%0p",arr);
$display("value of j : %0d",j);

arr.delete(1);
$display("arr : %0p",arr);

end
endmodule
```

## Usage of fixed size Array
```
class transaction;

rand bit [7:0] din;
randc bit [7:0] addr;
bit wr;
bit [7:0] dout;

constraint addr_c (addr > 10; addr < 18;);

endclass

class generator;

transaction t;
integer i;

task run();
for (i = 0; i < 100; i++)begin

t=new();
t.randomize();

end
endtask
endclass

class scoreboard;
bit [7:0] tarr[256] = '{default:0};
transaction t;
task run();

......................

if (t.wr == 1'b1) begin
tarr[t.addr] = t.din;
$display("[SCO]: Data stored din: "%0d addr: %0d", t.din,t.addr);
end

if (t.wr == 1'b0) begin
if(t.dout == 0)
$display ("[SCO]: No Data written at this location Test Passed");
else if(t.dout == tarr[t.addr])
$display("[SCO]: Valid Data found Test Passed");
else
$display(["SCO]: Test Failed");
end
end
endtask
endclass


```

##  Usage of Queue
```
class transaction;
rand bit [7:0]wdata;
bit [7:0] rdata;
rand bit wreq, rreq;

endclass

class generator;
transaction trans;
task run();
repeat(count) begin
trans = new();
assert(trans.randomize()) else ("Randomization Failed");

end
endtask
endclass

class scoreboard;
bit [7:0] rdata;
bit [7:0] queue [$];

task run;
.............
if(tr.wreq == 1)
begin
queue.push_front(tr.wdata);
end
else if(tr.rreq)
begin
if(tr.data != queue.popback())
$display("Data Mismatch at %0t,"$time);
end
endtask
endclass
```
