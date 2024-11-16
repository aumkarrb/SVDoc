- Randomization
- rand vs randc
- Constraint: Expression Vs Weighted Distribution
- Pre-randomization vs Post-Randomization
- Constraint Operators: Implication, Equivalence and IF ELSE
- Turning ON/OFF Constraint
- Building Transaction Class

## Understanding Generator
![[Pasted image 20241113135148.png]]
![[Pasted image 20241113143526.png]]
![[Pasted image 20241113143645.png]]

## Generating random Values with rand
```
/*
module dut(
input [3:0] a,b,
output [3:0] y
);

endmodule
*/
```

```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;
 
endclass

module tb; 
generator g;
int i = 0;
 
initial begin
g = new();

for (i = 0;i<10;i++) begin
g.randomize();
$display("value of a: %0d and b: %0d", g.a, g.b);
#10;

end
end

endmodule
```

## Checking Randomization is Successful: IF ELSE
```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;


constraint data { a > 15; }
 
endclass

module tb; 
generator g;
int i = 0;
int status 
initial begin
g = new();

for (i = 0;i<10;i++) begin
  if(!g.randomize()) begin
   $display("Randomization Failed at %0t",$time);
   $finish();
  end 


$display("value of a: %0d and b: %0d", g.a, g.b);
#10;

end
end

endmodule
```

## Checking Randomization is Successful: assert
```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;


//constraint data { a > 15; }
 
endclass

module tb; 
generator g;
int i = 0;
int status  = 0;
initial begin
g = new();

for (i = 0;i<10;i++) begin
  /*
  if(!g.randomize()) begin
   $display("Randomization Failed at %0t",$time);
   $finish();
  end 
  */
  
assert(g.randomise())else 
begin
   $display("Randomization Failed at %0t",$time);
   $finish();
end


$display("value of a: %0d and b: %0d", g.a, g.b);
#10;

end
end

endmodule
```

## Care while working with multiple Stimuli
```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;


//constraint data { a > 15; }
 
endclass

module tb; 
generator g;
int i = 0;
int status = 0;
initial begin


for(i = 0; i<10; i++) begin
g = new();
g.randomize()
$display("Value of a:%0d and b: %0d", g.a,g.b);
#!0;
end
end

endmodule
```

![[Pasted image 20241113190422.png]]

## Adding Constraint: Simple Expression
![[Pasted image 20241113205816.png]]

```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;
 
 /*
constraint data_a {a > 3; a < 7; }
constraint_data_b {b == 3}
*/

constraint data {a > 3; a < 7; b > 0;}

endclass

module tb; 
generator g;
int i = 0;
int status =0; 
initial begin


for(i = 0; i<10; i++) begin
g = new();
g.randomize()
$display("Value of a:%0d and b: %0d", g.a,g.b);
#!0;
end
end

endmodule
```

## Adding Constraint: Working with Ranges
![[Pasted image 20241113211842.png]]
```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;
 
/*
constraint data_a {a > 3; a < 7; }
constraint data_b {b == 3}
*/

//constraint data {a > 3; a < 7; b > 0;}

/*
constraint data {
                 a inside {[0:8],[10:11],15}; // 0 1 2 3 4 5 6 7 8 10 11 15
                 b inside {[3:11]}};          // 3 4 5 6 7 8 9 10 11
*/

constraint data {
!(a inside{[3:7]});
!(b inside{[5:9]});              
}

//// a = 3:7, b = 5:9

endclass

module tb; 
generator g;
int i = 0;
int status = 0;
initial begin


for(i = 0; i<10; i++) begin
g = new();
g.randomize()
$display("Value of a:%0d and b: %0d", g.a,g.b);
#!0;
end
end

endmodule
```

## External Function and Constraint
```
///// transaction
///// generator
class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;

extern constraint data; 

extern function void display();

endclass

constraint generator::data {
 a inside {[0:3]};
 b inside {[12:15]};
};

function void generator::display();
 $display("Value of a: %0d and b : %0d, a,b");
endfunction

module tb; 
 generator g;
 int i = 0;
 int status = 0;
 
 initial begin
   g = new();

   for(i = 0; i<15; i++) begin
      assert(g.randomize()) else $display("Randomization Failed");
      g.display();
     #10;
    end
end

endmodule
```

## Pre and Post Randomization Methods
```
//pre-randomize
//post-randomize

class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;
 int min;
 int max;
 
function void pre_randomize(input int min, input int max);
this.min = min; // refer to a class variable
this.max = max;
endfunction

constraint data {
a inside {[min:max]};
b inside {[min:max]};
};

function void post_randomize();
$display("value of a: %0d ad b: %0d", a,b);
endfunction

endclass

module tb; 
int i = 0;
generator g;
initial begin
g = new();

for(i = 0; i<10; i++) begin
g.pre_randomize(3,8);
g.randomize();
#10;
end
end

endmodule

```

## Things you need to consider while working with randc
![[Pasted image 20241114000647.png]]
```
//pre-randomize
//post-randomize

class generator; 
 
 randc bit [3:0] a, b; ///////// rand or randc 
 bit [3:0] y;
 int min;
 int max;
 
function void pre_randomize(input int min, input int max);
this.min = min; // refer to a class variable
this.max = max;
endfunction

constraint data {
a inside {[min:max]};
b inside {[min:max]};
};

function void post_randomize();
$display("value of a: %0d ad b: %0d", a,b);
endfunction

endclass

module tb; 
int i = 0;
generator g;
initial begin
g = new();

$display("SPACE 1");
g.pre_randomize(3,12);
for(i = 0; i<6; i++) begin
g.randomize();
#10;
end
$display("SPACE 2");
g.pre_randomize(3,12); // 3 4 5 6 7 8 9 10 11 12
for(i = 0; i<6; i++) begin
g.randomize();
#10;
end
end

endmodule
```

## Weighted Distribution
![[Pasted image 20241114011750.png]]
![[Pasted image 20241114012334.png]]
![[Pasted image 20241114012922.png]]

## Using Weighted Distribution
```
class first;

rand bit wr; // :=
rand bit rd: // :/

rand bit [1:0] var1;
rand bit [1:0] var2;

constraint data {
var1 dist {0 := 30, [1:3] := 90}; // 0 = 30/300 , 1,2,3 = 90/300
var2 dist {0 :/ 30, [1:3] :/ 90}; // 0,1,2,3, = 30/120 = 0.25


constraint cntrl {
wr dist {0 := 30, 1 := 70};
rd dist {0 :/ 30, 1 :/ 70};
}

endclass

module tb;

first f;

initial begin
f = new();

for (int i = 0, i<10, i++) begin
f.randomize();
$display("value of var1(:=) %0d and var2(:/) : "%0d", f.var1, f.var2);


end
end

endmodule
```

## Constraint Operator
![[Pasted image 20241114021022.png]]

1. Implication Operator
```
class generator; 

randc bit [3:0] a; 
rand bit ce;
rand bit rst;

constraint control_rst{
 rst dist {0 := 80, 1 := 20};
}

constraint control_ce{
 ce dist {1 := 80, 0 := 20};
}

constraint control_rst_ce{
(rst == 0 ) -> (ce == 1);
}

endclass

module tb;

generator g;

initial begin
g = new();

for (int i = 0; i<10; i++) begin
assert(g.randomize()) else $display("Randomization Failed");
$display("Value of rst: %0b and ce: %0b", g.rst, g.ce);

end
end
endmodule
```

2. Equivalence Operator
```
class generator;

randc bit [3:0] a;
rand bit wr; //write to mem
rand bit oe; //output enable

constraint wr_c {
wr dist {0:=50, 1:=50};
}

constraint oe_c {
oe dist {1:=50, 0:=50};
}

constraint wr_oe_c{
(wr == 1) <-> (oe == 0);
}

endclass

module tb;
generator g;

initial begin
g = new();

for(int i = 0; i<10; i++) begin
assert(g.randomize()) else $display("Randomization Failed");
$display("value of wr : %0b and oe: %0b", g.wr, g.oe);
end
end
endmodule

```

3. IF ELSE Operator
```
class generator;

randc bit [3:0] raddr, waddr;
rand bit wr; ///write to mem
rand bit oe; ///output enable

constraint wr_c {
wr dist {0:=50, 1:=50};
}

constraint oe_c {
oe dist {1:=50, 0:=50};
}

constraint wr_oe_c {
( wr == 1) <-> (oe == 0);
}

constraint write {
if (wr == 1)
{
 waddr inside {[11:15]};
 raddr == 0;
}

}

endclass

module tb;
generator g;
initial begin
g = new();
for(int i = 0; i<15; i++)begin
assert(g.randomze()) else $display("Randomization Failed");
$display("Value of wr : %0b | oe : %0b | raddr : %0d | waddr: %0d|", g.wr,g.oe,g.raddr,g.waddr);
end
end
endmodule
```

## Turning ON and OFF Constraint
```
class generator;

randc bit [3:0] raddr, waddr;
rand bit wr; /// write to mem
rand bit oe; ///output enable

constraint wr_c{
wr dist {0:=50, 1:=50};
}

constraint oe_c{
oe dist {1:=50, 0:=50};
}

constraint wr_oe_c{
(wr == 1) -> (oe == 0);
}

endclass

module tb;

generator g;

initial begin
g = new();

g.wr_oe_c.constraint_mode(0); //// 1 -> constraint is on // 0 -> constraint is off
if(g.oe_c.constraint_mode(0))
  $display("Constraint Status oe_c : %0d",g.wr_oe_c.constraint_mode());
for(int i = 0; i<20; i++) begin
  assert(g.randomize()) else $display("Randomization Failed");
  $display("value of wr: %0b | oe: %0b | ", g.wr, g.oe);
end
end
endmodule
```

## Understanding FIFO DUT
![[Pasted image 20241114092328.png]]

![[Pasted image 20241114092518.png]]

```
module fifo(
input wreq, rrreq,
input clk,
input rst,
input [7:0] wdata,
input [7:0] rdata,
output f,e
);

endmodule
```

```
class transaction;

bit clk;
bit rst; 

rand bit wreq, rreq: /// Active

rand bit [7:0] wdata; 
bit [7:0] rdata;

bit empty; 
bit full; 

constraint ctrl_wr{
wreq dist {0:=30; 1:=70;};
}

constraint ctrl_rd {
rreq dist {0:=30; 1:= 70;};
}

constraint wr_rd{
wreq != rreq;
}

endclass
```