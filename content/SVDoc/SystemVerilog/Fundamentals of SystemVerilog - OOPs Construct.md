- Understanding Class
- Adding Method in Class
- Method arguments: Pass by Value, Pass by Reference
- Custom Constructor
- Shallow Copy Vs Deep Copy
- Inheritance
## Fundamentals of Class
in SystemVerilog
```
class first;

reg [2:0] data;
reg [1:0] data2; // Data Members
endclass

module tb;
first f; // serves as handler

initial begin
f = new(); // serves as a constructor
f.data = 3'b010;
f.data2 = 2'b10;
f = null; // deallocates memory assigned to a class
#1;
$display("Value of data: %0d and data2 : %0d," f.data, f.data2);
end

f.data;
f.data2;

endmodule
```

in Verilog
```
module top(
input a,b,
output y
);

reg temp;
assign y = a & b;
endmodule

endmodule
module top2(
input c,d,
output y
)

top dut (c,d,y);
dut.temp;
endmodule
```

## Ways to add Method to Class
```
task sys_rst();
rst <=1'b1;
#30;
rst <=1'b0;
endtask
```

![[Pasted image 20241111134809.png]]

## Using Function
```
module tb;

bit [4:0] res = 0;
bit [3:0] ain = 4'b0100;
bit [3:0] bin = 4'b0010;

//function bit [4:0] add(input bit [3:0] a = 4'b0100, b=4'b0010);
function bit [4:0] add();
return ain + bin;
//return a + b;
endfunction

function void display_ain_bin();
$display("Value of ain: %0d and bin : %0d",ain,bin);
endfunction

//bit [4:0] res = 0;
//bit [3:0] ain = 4'b0100;
//bit [3:0] bin = 4'b0010;

initial begin
//res = add(4'b0100, 4'b0010);
//res = add(ain,bin);
//res = add();
display_ain_bin();
//$display("value of addition: %0d",res);


end
endmodule
```

## Using Task
```
module tb;


////default direction: input

/*
task add(input bit [3:0] a, input bit [3:0] b, output bit [4:0] y);
y = a + b ;
endtask
*/

bit [3:0] a,b;
bit [4:0] y;

bit clk = 0;

always #5 clk = ~clk ////20ns --> 50 Mhz

task add(input bit [3:0] a, input bit [3:0] b, output bit [4:0] y);
y = a + b ;
$display("a: %0d and b = %0d and y : %0d",a,b, y);
endtask

task stim_a_b();
a = 1;
b = 3;
add();
#10;
a = 5;
b = 6;
add();
#10;
a = 7;
b = 8;
add()
#10;
endtask

task stim_clk ();
@(posedge clk); //wait
a = $urandom();
b = $urandom();
add();
endtask

initial begin
#110;
$finish();
end


initial begin
//stim_a_b();
//end 
for (int i = 0; i<11; i++) begin
stim_clk();
end
end

//a = 7;
//b = 7;
//add(a,b,y);
//add();
//$display("Value of y : %0d", y");

//end
endmodule
```
-> In task, you can add timing control, which is not possible in Function.

## Understanding Pass by Value
![[Pasted image 20241111194912.png]]

![[Pasted image 20241111201006.png]]
![[Pasted image 20241111201107.png]]
Everytime you create a function with some parameter its always treated like a copy variable.
# Demonstration of Pass by Value
```
module tb;

task swap (input bit [1:0] a, b);
bit [1:0] temp;

temp = a;
a = b;
b = temp;

$display("Value of a : %0d and b : %0d", a,b);
endtask

bit [1:0] a;
bit [1:0] b;

initial begin
a = 1;
b = 2;
swap(a,b);
$display("Value of a : %0d and b : %0d", a,b);
end

endmodule

```

# Demonstration of Pass by Reference
```
module tb;

task automatic swap (const ref bit [1:0] a, ref bit [1:0] b); 
/// function automatic bit [1:0] add (argument);
bit [1:0] temp;

temp = a;
//a = b;
b = temp;

$display("Value of a : %0d and b : %0d", a,b);
endtask

bit [1:0] a;
bit [1:0] b;

initial begin
a = 1;
b = 2;
swap(a,b);
$display("Value of a : %0d and b : %0d", a,b);
end

endmodule
```


| Function                                                                                                                           | Task                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Do not support timing control function                                                                                             | Support Timing Control                                                 |
| Multiple input single return value                                                                                                 | Multiple input / output                                                |
| use ref, automatic for array                                                                                                       | use ref, automatic for array                                           |
| Common Usage: printing values of Data Member, initializing values of variable, Time independent expression, return data from class | Common usage: Time dependent expression, Scheduling processes in class |

## Using Array in a Function
```
module tb;

bit [3:0] res[16];

function automatic void init_arr(ref [3:0] a[16]);
 for(int i = 0; i <=15; i++) begin
 a[i] = i;
 end
endfunction

initial begin
init_arr(res);
for(int i = 0; i <=15; i++) begin
$display("res[%0d] : %0d", i, res[i]);
 res[i] = i;
 end
end

endmodule
```

## User defined Constructor
```
class first;

int data;

function new(input int datain = 0);
data = datain;
//data = 32;
endfunction
endclass

module tb;
first f1;

initial begin
f1 = new(23);
$display("Data: %0d, f1.data");

end

endmodule
```

## Multiple arguments to a Constructor
```
class first;

int data1;
bit [7:0] data2;
shortint data3;

function new(input int data1 = 0, input bit [7:0] data2 = 8'h00, input shortint data3 = 0);
this.data1 = data1;
this.data2 = data2;
this.data3 = data3;
endfunction


endclass

module tb;

first f1;

initial begin
// f1 = new(23,4,35); /// follow position
f1 = new(.data2(4), .data3(5), .data1(23) );
$display("Data1 : %0d, Data2: %0d and Data3 : %0d", f1.data1, f1.data2, f1.data3);

end

endmodule
```

## Using task in Class
```
class first;

int data1;
bit [7:0] data2;
shortint data3;

function new(input int data1 = 0, input bit [7:0] data2 = 8'h00, input shortint data3 = 0);
this.data1 = data1;
this.data2 = data2;
this.data3 = data3;
endfunction

task display();
$display("value of Data1: %0d, Data2 : %0d and Data3 : %0d", data1, data2, data3);
endtask

endclass

module tb;

first f1;

initial begin
// f1 = new(23,4,35); /// follow position
f1 = new(.data2(4), .data3(5), .data1(23) );
f1.display();

end

endmodule
```

## Using Class in Class
```
class first;
int data = 34;
task display();
$display("Value of data: %0d", data);
endtask
endclass

class second; 
first f1: 
function new();
f1 = new();
endfunction
endclass

module tb;
second s;
initial begin
s = new();
$display("Value of Data : %0d", s.f1.data);
s.f1.display();

s.f1.data = 45; 
s.f1.display();

end

endmodule
```

## Scope of Data Member
```
class first;
local int data = 34;

task set(input int data);
this.data = data;
endtask

function int get();
return data;
endfunction


task display();
$display("Value of data: %0d", data);
endtask
endclass

class second; 
first f1: 
function new();
f1 = new();
endfunction
endclass

module tb;
second s;
initial begin
s = new();
s.f1.set(123);
$display("Value of data : %0d", s.f1.get() );

end

endmodule
```

## Copying Object
```
class first;

int data;

endclass

module tb;
first f1;
first p1;


initial begin
f1 = new();
////////////////// ///1. constructor
f1.data = 24;
////////////////// ///2. processing
p1 = new f1;
/////////////////  ///3. copying data from f1 to p1

$display("value of data member : %0d", p1.data); ///4. processing

p1.data = 12;
$display("value of data member : %0d", f1.data); ///4. processing

end
endmodule
```

## Strategies to copy Object
![[Pasted image 20241111234150.png]]

# Custom Method
```
class first;
int data = 34;
bit [7:0] temp = 8'h11;

function first copy();
copy = new();
copy.data = data;
copy.temp = temp;
endfunction

endclass

module tb;

first f1;
first f2;

initial begin
f1 = new();
f2 = new();

f2 = f1.copy;
$display("Data : %0d and TEMP : %0x", f2.data, f2.temp);

end

/*
initial begin
f1 = new();
//////////
f1.data = 45;
//////////
f2 = new f1;
//////////
f2.data = 56;
*/


$display("%0d", f1.data);



end

endmodule
```

## Understanding Shallow Copy
![[Pasted image 20241112012930.png]]

# Demonstration
```
class first;

int data = 12; 

endclass

class second;

int ds = 34;
first f1;

function new();
f1 = new();
endfunction

function second copy();
copy = new();
copy.ds = ds;
copy.f1 = 

endfunction
endclass


module tb;

second s1, s2;

initial begin
s1 = new();
s1.ds = 45;
s2 = new s1;

$display(" value of ds: %0d", s2.ds);

s2.ds = 78;

$display(" value of ds: %0d", s1.ds);

s2.f1.data = 56;

$display("value of data: %0d", s1.f1.data);


end

endmodule
```


## Understanding Deep Copy

![[Pasted image 20241112015046.png]]


# Demonstration

```
class first;

int data = 12; 
function first copy();
copy = new();
copy.data = data;
endfunction
 

endclass

class second;

int ds = 34;
first f1;

function new();
f1 = new();
endfunction

function second copy();
copy = new();
copy.ds = ds;
copy.f1 = f1.copy
endfunction

endclass

module tb;

second s1, s2;

initial begin
s1 = new();
s2 = new();

s1.ds = 45;

s2 = s1.copy();

$display("value of s2: %0d", s2.ds);

s2.ds = 78;

$display("value of s1: %0d", s1.ds);

s2.f1.data = 98;

$display("value of s1: %0d", s1.f1.data);

end

endmodule
```

Only Data Members ---> Create Custom Copy Method ---> Task

DM + Other Class Instance ----> Shallow Copy ---->  Copy of DM + Class handlers for both                                                                                                                         original as well as copy remain                                                                                                               same
DM + Other Class Instance ----> Deep Copy ----> Copy of DM + Independent handler for the class

## Extending Class Properties by Inheritance
```
class first; ///parent class

int data = 12; 
function void display();
$display("value of data : %0d", data);
endfunction
endclass

class second extends first; ///child class
int temp = 34; 
function void add();
$display("value after processing : %0d", temp+4);
endfunction
endclass

module tb; 
second s; 

initial begin
s = new();
$display("value of data: %0d", s.data);
s.display();
$display("value of temp: %0d", s.temp);
s.add()

end
endmodule
```

## Polymorphism
```
class first;  ///parent class

int data = 12;

virtual function void display();
$display("FIRST: value of data: %0d", data);
endfunction

endclass


class second extends first; //child class
int temp = 34; 

function void add();
$display("second value after process: %0d" , temp+4);
endfunction

function void display();
$display("SECOND: Value of data: %0d", data);
endfunction


module tb;
first f;
second s; 

initial begin
f = new();
s = new();

f = s;
f.display();

end
endmodule
```

## Understanding Usage of Super Keyword
![[Pasted image 20241113122928.png]]
```
class first; ////// parent class
int data;

function new(input int data);
this.data = data; 
endfunction


endclass

class second extends first;
int temp;
function new(int data, int temp);
super.new(data);
this.temp = temp;
endfunction

endclass

module tb;
second s; 
initial begin
s = new(67, 45);
$display("value of data: %0d and temp: %0d", s.data, s.temp);


end
endmodule
```
