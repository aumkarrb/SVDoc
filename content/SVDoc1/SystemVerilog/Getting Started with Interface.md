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