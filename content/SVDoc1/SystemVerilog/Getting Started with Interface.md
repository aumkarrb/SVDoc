- Interface
- MODPORT
- accessing interface in Class and Testbench top
- adding value to Interface
- Creating Generator and Driver Class
- Creating Monitor and Scoreboard Class

## Interface
![[Pasted image 20241117015716.png]]

## Adding Interface to Simple RTL

   Design of an Adder
```
module add(
input [3:0] a,b,
output [4:0] sum

);


assign sum = a + b;

endmodule
```
```
interface add_if;
 logic [3:0] a;
 logic [3:0] b;
 logic [4:0] sum; 
endinterface

module tb;

and_if aif();

//add dut (aif.a, aif.b, aif.sum);  ////Positional Map
add dut (.b(aif.b), .a(aif.a), .sum(aif.sum)); ///mapping by name

initial begin
aif.a = 4;
aif.b = 4;
#10;
aif.a = 3;
#10;
aif.b = 7;
end

initial begin
$dumpfile("dump.vcd");
$dumpvars;

endmodule
```
## Using blocking operator for Interface Variables
```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```


```
interface add_if; 
 logic [3:0] a;
 logic [3:0] b; 
 logic [4:0] sum; 
 logic clk; 
endinterface

module tb;

add_if aif();

//add dut (aif.a, aif.b, aif.sum); //positional map
add dut(.b(aif.b), .a(aif.a), .sum(aif.sum), .clk(aif.clk)); //mapping by name

initial begin
aif.clk = 0;
end


always #10 aif.clk = ~aif.clk;

initial begin
aif.a = 1;
aif.b = 5; 
#22;
aif.a = 3;
#20;
aif.a = 4; 
#8
aif.a = 5; 
#20;
aif.a = 6;
end

initial begin
$dumpfile("dump.vcd");
$dumpvars;
#100;
$finish();
end

endmodule

```
![[Pasted image 20241117152808.png]]

## Using Non-Blocking Operator for Interface Variables

```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule

```

```
interface add_if; 
 logic [3:0] a;
 logic [3:0] b; 
 logic [4:0] sum; 
 logic clk; 
endinterface

module tb;

add_if aif();
//add dut (aif.a, aif.b, aif.sum); //positional map
add dut(.b(aif.b), .a(aif.a), .sum(aif.sum), .clk(aif.clk)); //mapping by name

initial begin
aif.clk = 0;
end

always #10 aif.clk = ~aif.clk;

initial begin
aif.a = 1;
aif.b = 5; 
repeat(3) @(posedge aif.clk);
#22;
aif.a = 3;
#20;
aif.a = 4; 
#8
aif.a = 5; 
#20;
aif.a = 6;
end

initial begin
$dumpfile("dump.vcd");
$dumpvars;
#100;
$finish();
end

endmodule

```

## Why we prefer LOGIC and WIRE over REG in Interface
```
module add(
input [3:0] a,b,
output [4:0] sum

);


assign sum = a + b;

endmodule
```

```
interface add_if;
 wire [3:0] a;
 wire [3:0] b;
 wire [4:0] sum; 
 
endinterface
////// interface with all reg type, then we are not allowed to connect variable in interface to the output port of DUT
///// interface with all wire type, we are not allowed to apply stimulus using initial or always

module tb;

add_if aif();

add dut (aif.a, aif.b, aif.sum);  ////Positional Map

initial begin
aif.a = 1;
aif.b = 3;


end

initial begin
$dumpfile("dump.vcd");
$dumpvars;

endmodule
```
## Adding Driver Code to Interface
```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```
```
interface add_if;
logic [3:0] a,b;
logic [4:0] sum;
logic clk;
endinterface

class driver;

virtual add_if aif;

task run();
forever begin
 @(posedge aif.clk);
aif.a <= 3;
aif,b <= 3;
end
endtask

endclass


module tb;
add_if aif();

add dut(.a(aif.a),.b(aif.b), .sum(aif.sum), .clk(aif.clk));

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

driver drv;

initial begin
drv = new();
drv.aif = aif;
drv.run();
end

initial begin
$dumpfile("dump.vcd");
$dumpvars;
#100;
$finish();
end
endmodule
```
## Understanding MODPORT

```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```

```
interface add_if; 
logic [3:0] a; 
logic [3:0] b; 
logic [4:0] sum; 
logic clk; 

modport DRV (output a,b, input sum,clk);



endinterface

class driver; 

virtual add_if aif;

task run();
 forever begin
  @(posedge aif.clk);
  aif.a <= 2;
  aif.b <= 3;
  $display("[DRV] : Interface Trigger");
 end
 endtask
 
endclasss

module tb;

add_if aif();
driver drv;

add dut (aif.a, aif.b, aif.sum, aif.clk );

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

initial begin
drv = new();
drv.aif = aif; 
drv.run();

end

initial begin
$dumpfile("dump.vcd");
$dumpvars;
#100;
$finish
end 
endmodule

```

## Adding Generator
```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```
```
////1.add transaction constructor in generator custom constructor 
////2.Send deep copy of transaction between generator and driver 


class transaction;

 randc bit [3:0] a; 
 randc bit [3:0] b;
 bit [4:0] sum;

function void display();
 $display("a : %0d \t b: %0d \t sum : %0d",a,b,sum)
endfunction

function transaction copy();
copy = new();
copy.a = this.a;
copy.b = this.b;
copy.sum = this.sum;
endfunction

endclass


class generator; 

 transaction trans;
 mailbox #(transaction) mbx;
 event done;
 
function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
 trans = new();
endfunction

function transaction copy();

task run();
  //trans = new();
 for(int i = 0; i<10, i++) begin
     trans.randomize(); 
     mbx.put(trans.copy);
     $display("[GEN] : DATA SENT TO DRIVER");
     trans.display();
     #20;
  end
  -> done;
  endtask


endclass

interface add_if; 
logic [3:0] a; 
logic [3:0] b; 
logic [4:0] sum; 
logic clk; 
endinterface

class driver; 

virtual add_if aif;
mailbox #(transaction) mbx;
transaction data; 
event next; 

function new(mailbox #(transaction) mbx);
 this.mbx = mbx; 
endfunction
 

task run();
 forever begin
 mbx.get(data);
  @(posedge aif.clk);
  aif.a <= data.a;
  aif.b <= data.b;
  $display("[DRV] : Interface Trigger");
  data.display();
 end
 endtask
 
endclasss

module tb;

add_if aif();
driver drv;
generator gen;
event done;

mailbox #(transaction) mbx;

add dut (aif.a, aif.b, aif.sum, aif.clk );

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

initial begin
 mbx = new();
 drv = new(mbx); 
 gen = new(mbx);
 drv.aif = aif;
 done = gen.done;
end

initial begin 
fork
  gen.run();
  drv.run();
join_none
 wait(done.triggered);
 $finish();
end

initial begin
$dumpfile("dump.vcd");
$dumpvars;
end 
endmodule

```

## Injecting Error

![[Pasted image 20241118233437.png]]
```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```
```
////1.add transaction constructor in generator custom constructor 
////2.Send deep copy of transaction between generator and driver 


class transaction;

 randc bit [3:0] a; 
 randc bit [3:0] b;
 bit [4:0] sum;

function void display();
 $display("a : %0d \t b: %0d \t sum : %0d",a,b,sum)
endfunction

endclass

virtual function transaction copy();
copy = new();
copy.a = this.a;
copy.b = this.b;
copy.sum = this.sum;
endfunction

endclass

class error extends transaction; 

 // constraint data_c (a == 0; b == 0;)

virtual function transaction copy();
copy = new();
copy.a = 0;
copy.b = 0;
copy.sum = this.sum;
endfunction

endclass

class generator; 

 transaction trans;
 mailbox #(transaction) mbx;
 event done;
 
function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
 trans = new();
endfunction

function transaction copy();

task run();
  //trans = new();
 for(int i = 0; i<10, i++) begin
     trans.randomize(); 
     mbx.put(trans.copy);
     $display("[GEN] : DATA SENT TO DRIVER");
     trans.display();
     #20;
  end
  -> done;
  endtask


endclass

interface add_if; 
logic [3:0] a; 
logic [3:0] b; 
logic [4:0] sum; 
logic clk; 
endinterface

class driver; 

virtual add_if aif;

mailbox #(transaction) mbx;

transaction data; 
event next; 

function new(mailbox #(transaction) mbx);
 this.mbx = mbx; 
 trans = new();
endfunction
 

task run();
	 forever begin
 mbx.get(data);
  @(posedge aif.clk);
  aif.a <= data.a;
  aif.b <= data.b;
  $display("[DRV] : Interface Trigger");
  data.display();
 end
 endtask
 
endclasss

module tb;

add_if aif();
driver drv;
generator gen;
event done;
 error err;
 

mailbox #(transaction) mbx;

add dut (aif.a, aif.b, aif.sum, aif.clk );

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

initial begin
 mbx = new();
 err = new();
 drv = new(mbx); 
 gen = new(mbx);
 
 gen.trans = err;
 
 drv.aif = aif;
 done = gen.done;
end

initial begin 
fork
  gen.run();
  drv.run();
join_none
 wait(done.triggered);
 $finish();
end

initial begin
$dumpfile("dump.vcd");
$dumpvars;
end 
endmodule
```
## Adding Monitor and Scoreboard
```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```

```
////1.add transaction constructor in generator custom constructor 
////2.Send deep copy of transaction between generator and driver 


class transaction;

 randc bit [3:0] a; 
 randc bit [3:0] b;
 bit [4:0] sum;

function void display();
 $display("a : %0d \t b: %0d \t sum : %0d",a,b,sum)
endfunction

endclass

interface add_if; 
logic [3:0] a; 
logic [3:0] b; 
logic [4:0] sum; 
logic clk; 
endinterface

class monitor;
 mailbox #(transaction) mbx; 
 transaction trans;
 virtual  add_if aif;

 function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
endfunction

task run();
trans = new();
 forever begin
 @(posedge aif.clk);
 trans.a = aif.a; 
 // Operator used to add value to data member of transaction must be "="
 trans.b = aif.b; 
 trans.sum = aif.sum; 
 $display("[MON]: DATA SENT TO SCOREBOARD");
 trans.display();
 mbx.put(trans)
end
endtask
endclass

class scoreboard;
 mailbox #(transaction) mbx; 
 transaction trans;

 function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
endfunction

task run();
 forever begin 
  mbx.get(trans);
  $display("[SCO] : DATA RCVD FROM MONITOR")
  trans.display();
  #20;
 end
endtask


endclass

module tb;
add_if aif();
monitor mon;
scoreboard sco;
mailbox #(transaction) mbx;


add dut (aif.a, aif.b, aif.sum, aif.clk );

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

initial begin
 for (int i = 0; i < 20; i++) begin
  @(posedge aif.clk);
  aif.a <= $urandom_range(0,15);
  aif.b <= $urandom_range(0,15);
 end
end

initial begin
mbx = new();
mon = new(mbx);
sco = new(mbx);
mon.aif = aif;
end

initial begin 
fork 
mon.run();
sco.run();
join
end

initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #450; 
  $finish();

end
endmodule
```
## Tweaking Monitor and Scoreboard Code
```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```
```
////1.add transaction constructor in generator custom constructor 
////2.Send deep copy of transaction between generator and driver 


class transaction;

 randc bit [3:0] a; 
 randc bit [3:0] b;
 bit [4:0] sum;

function void display();
 $display("a : %0d \t b: %0d \t sum : %0d",a,b,sum)
endfunction

endclass

interface add_if; 
logic [3:0] a; 
logic [3:0] b; 
logic [4:0] sum; 
logic clk; 
endinterface

class monitor;
 mailbox #(transaction) mbx; 
 transaction trans;
 virtual  add_if aif;

 function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
endfunction

task run();
trans = new();
 forever begin
 repeat(2)@(posedge aif.clk);
 trans.a = aif.a; 
 // Operator used to add value to data member of transaction must be "="
 trans.b = aif.b; 
 trans.sum = aif.sum; 
 $display("[MON]: DATA SENT TO SCOREBOARD");
 trans.display();
 mbx.put(trans)
end
endtask
endclass

class scoreboard;
 mailbox #(transaction) mbx; 
 transaction trans;

 function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
endfunction

task run();
 forever begin 
  mbx.get(trans);
  $display("[SCO] : DATA RCVD FROM MONITOR")
  trans.display();
  #40;
 end
endtask


endclass

module tb;
add_if aif();
monitor mon;
scoreboard sco;
mailbox #(transaction) mbx;


add dut (aif.a, aif.b, aif.sum, aif.clk );

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

initial begin
 for (int i = 0; i < 20; i++) begin
  repeat(2)@(posedge aif.clk);
  aif.a <= $urandom_range(0,15);
  aif.b <= $urandom_range(0,15);
 end
end

initial begin
mbx = new();
mon = new(mbx);
sco = new(mbx);
mon.aif = aif;
end

initial begin 
fork 
mon.run();
sco.run();
join
end

initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #450; 
  $finish();

end
endmodule
```

## Adding Simple Scoreboard Model

```
module add(
input [3:0] a,b,
output reg [4:0] sum, 
input clk
);

always @(posedge clk)
 begin
   sum <= a + b; 
   end
endmodule
```
```
////1.add transaction constructor in generator custom constructor 
////2.Send deep copy of transaction between generator and driver 


class transaction;

 randc bit [3:0] a; 
 randc bit [3:0] b;
 bit [4:0] sum;

function void display();
 $display("a : %0d \t b: %0d \t sum : %0d",a,b,sum)
endfunction

endclass

interface add_if; 
logic [3:0] a; 
logic [3:0] b; 
logic [4:0] sum; 
logic clk; 
endinterface

class monitor;
 mailbox #(transaction) mbx; 
 transaction trans;
 virtual  add_if aif;

 function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
endfunction

task run();
trans = new();
 forever begin
 repeat(2)@(posedge aif.clk);
 trans.a = aif.a; 
 // Operator used to add value to data member of transaction must be "="
 trans.b = aif.b; 
 trans.sum = aif.sum; 
 $display("-----------------------------------------------");
 $display("[MON]: DATA SENT TO SCOREBOARD");
 trans.display();
 mbx.put(trans)
end
endtask
endclass

class scoreboard;
 mailbox #(transaction) mbx; 
 transaction trans;

 function new(mailbox #(transaction) mbx);
 this.mbx = mbx;
endfunction

task compare(input transaction trans);
if((trans.sum) == (trans.a + trans.b))
$display("[SCO]: SUM RESULT MATCHED");
else
$error ("[SCO]: SUM RESULT MISMATCHED"); //$warning, $fatal

endtask

task run();
 forever begin 
  mbx.get(trans);
  $display("[SCO] : DATA RCVD FROM MONITOR")
  trans.display();
  compare(trans);
  $display("-----------------------------------------------");
  #40;
 end
endtask


endclass

module tb;
add_if aif();
monitor mon;
scoreboard sco;
mailbox #(transaction) mbx;


add dut (aif.a, aif.b, aif.sum, aif.clk );

initial begin
aif.clk <= 0;
end

always #10 aif.clk <= ~aif.clk;

initial begin
 for (int i = 0; i < 20; i++) begin
  repeat(2)@(posedge aif.clk);
  aif.a <= $urandom_range(0,15);
  aif.b <= $urandom_range(0,15);
 end
end

initial begin
mbx = new();
mon = new(mbx);
sco = new(mbx);
mon.aif = aif;
end

initial begin 
fork 
mon.run();
sco.run();
join
end

initial begin
  $dumpfile("dump.vcd");
  $dumpvars;
  #450; 
  $finish();

end
endmodule
```