## Learning Path

![[Pasted image 20241027225027.png]]
## Type of DUT signals

![[Pasted image 20241028005945.png]]
Global Signal - initial block is extremely useful
--> Will be utilised in testbench block(stimulus generation)
--> rest others will be utilised in the verification environment

 
## Format of Initial Block in Testbench

![[Pasted image 20241028010533.png]]
--> Testbench module do not have ay input and output ports.

![[Pasted image 20241028010850.png]]

-->  initial start execution at 0nsec (can have single or multiple statements)
--> reg holds the value


```
`timescale 1ns/1ps
  module tb;
  
  reg a = 0;
  
  initial a = 1;
  
  initial begin
   
    a = 1;
    #10;
    a = 0;
    
endmodule
```

## Usage of Initial Block

```
module tb();
///global signal  clk, rst
  reg  clk; //defining a variable
  reg  rst;
  
  reg[3:0] temp; //generating random waveform for muliple signal
  
  
  initial begin //1. initialising a  GLOBAL variable
    clk = 1'b0; //generating a single bit value which 0 in binary format
    rst = 1'b0;
  end
  //2. Random signal for data/ control
    initial begin
     rst = 1'b1;
     #30; 
     rst = 1'b0;
     end
    
    initial begin
    temp = 4'b0100; //4 Bits of temp
    #10;
    temp = 4'b1100;
    #10;
    temp = 4'b0011;
    #10;
    end
  
   //3. System Task at the start of simulation 
  initial begin //as a control statement
    $dumpfile("dump.vcd");
    $dumpvars;
  end
  
  //send the values on console whenever there are changes in the value
  //4. Analyzing values of variable on console
  initial begin
    $monitor("Temp : %0d at time: %0t", temp, $time); 
   end
  //5. Stop simulation by forcefully calling $finish
  initial begin
    #200;
    $finish();
    end
  
endmodule
```
--> Initialized Variable
--> Generate Reset Signal
--> Executing System Task
## Format of always block

```
'timescale 1ns / 1ps
module tb();

always@() // always_comb(combinational), always_ff(sequential), always_latch(latch)
// for testbench code, it's always
always statement;
always begin
..................
..................
..................

end
endmodule
```

![[Pasted image 20241028165823.png]]   
y = a & b , Comb --> all inputs,  Sequential --> Clock

```
always@(a,b)              always@(posedge clk)
```

## Usage of always block 
![[Pasted image 20241029233753.png]]

```
`timescale 1ns/1ps

module tb();

reg clk; //x
reg rst;
reg clk50 = 0;
reg clk25 = 0;

initial begin
clk = 1'b0;
rst = 1'b0;
clk50 = 0;

end

always #5 clk = ~clk; //100 Mhz
always #10 clk50 = ~clk50; //50 Mhz
always #20 clk25 = ~clk25; //25 Mhz

initial begin
$dumpfile("dump.vcd");
$dumpvars;
end

initial begin
#200;
$finish();
end

endmodule
```
--> Generate Clock Signal 
## Aligning reference of the generated clock and reference clock 
```
`timescale 1ns/1ps

module tb();

reg clk; //x
reg rst;
reg clk50 = 0;
reg clk25 = 0;

initial begin
clk = 1'b0;
rst = 1'b0;
clk50 = 0;

end

/*
always #5 clk = ~clk; //100 Mhz
always #10 clk50 = ~clk50; //50 Mhz
always #20 clk25 = ~clk25; //25 Mhz
*/

always #5 clk = ~clk;
always begin
#5; 
clk50 = 1;
#10;
clk50 = 0;
#5;
end

always begin
#5;
clk25 = 1;
#20;
clk25 = 0;
#15;

initial begin
$dumpfile("dump.vcd");
$dumpvars;
end

initial begin
#200;
$finish();
end

endmodule
```

--> Both have their edge aligned
## Understanding 'timescale directive


![[Pasted image 20241101191631.png]]![[Pasted image 20241101192208.png]]

```
`timescale 1ns / 1ns //1 -> 10^0
                    // 10^3 -> 1

module tb();



reg clk16 = 0;
reg clk8 = 0; //initialize variable

always #31.25 clk16 = clk16;
always #62.5 clk8 = ~clk8;

initial begin
$dumpfile("dump.vcd");
$dumpvars;
end

initial begin
#200
$finish();
end

endmodule
```

## Understanding parameters for generating Clock

![[Pasted image 20241102180228.png]]

```
`timescale 1ns/ 1ps

module tb();

reg clk = 0;
reg clk50 = 0;

always #5 clk = ~clk //100 Mhz

real phase = 10;
real ton = 5;
real toff = 5;

initial begin
#phase; 
while(1) begin
clk50 = 1;
#ton;
clk50 = 0;
#toff;
end
end 

initial begin
#200;
$finish();
end

endmodule
```

Another way using ```task ``` 

```
`timescale 1ns/ 1ps

module tb();

reg clk = 0;
reg clk50 = 0;

always #5 clk = ~clk //100 Mhz

/*
real phase = 10;
real ton = 5;
real toff = 5;
*/*

task clkgen(input real phase, input real ron, input real toff);
#phase; 
while(1) begin
clk50 = 1;
#ton;
clk50 = 0;
#toff;
end
endtask

initial begin
clkgen(7,5,5);
end

initial begin
#200;
$finish();
end

endmodule
```

```
`timescale 1ns / 1ps
module tb()'

reg clk = 0;
reg clk50 = 0;

always #5 clk = ~clk; //100 Mhz
/* 
real phase = 10;
real ton = 5;
real toff = 5;
*/

/*
task clkgen(input real phase, input real ton, input real toff);
#phase;
while(1) begin
clk50 = 1;
#ton;
clk50 = 0;
#toff;
end
endtask 
*/

task calc (input real freq_hz, input real duty_cycle, input real phase,
output real pout, output real ton, output real toff);
pout = phase;
ton = (1000_000_000 / frewq_hz) - ton;
endtask

task clkgen(input real phase, input real ton, input real toff);
@(possedge clk);
#phase;
while(1) begin
clk50 = 1;
#ton;
clk50 = 0;
#toff;
end 
endtask

real phase;
real ton;
real toff;

initial begin 
calc(100_000_000, 0.1,2,phase,ton,toff)
clkgen(phase,ton,toff);
end

initial begin
#200;
$finish();
end
endmodule


```