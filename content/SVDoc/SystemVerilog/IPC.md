- Interprocess Communication
- FORK JOIN
- EVENT
- SEMAPHORE
- MAILBOX
- Parameterized MAILBOX
## Interprocess Communication Mechanism
![[Pasted image 20241114192100.png]]

## Events
```
/// Trigger : -> 
/// edge sensitive_blocking @(), level_sensitive_non_blocking wait()

module tb;
event a;

initial begin
#10; 
-> a;
end

initial begin
//@(a); 
wait(a.triggered);
$display("Received Event %0t", $time);

end

endmodule
```

## @ vs Wait
```
/// Trigger : -> 
/// edge sensitive_blocking @(), level_sensitive_non_blocking wait()

module tb; 
event a1,a2;

initial begin
-> a1;
-> a2;
end

initial begin
//@(a1);
wait(a1.triggered);
$display("Event  A1 Trigger");
//@(a2);\
wait(a2.triggered);
$display("Event A2 Trigger");
end

endmodule
```

## Executing Multiple Processes
![[Pasted image 20241115210856.png]]

```
module tb; 
int data1, data2;
event done;

int i = 0;

////Generator
initial begin
  for(i = 0; i < 10;i++) begin
  data1 = $urandom();
  $display("Data Sent: %0d", data1);
  #10;
  end
 -> done ; 
end

////Driver
initial begin
 forever begin
  #10;
  data2 = data1;
  $display("Data RCVD: %0d", data2);
   end 
end
//////////////////

initial begin
wait(done.triggered);
$finish();
end

endmodule
```

## Multiple Process with FORK JOIN
```
 module tb;
      int i =0;
      bit [7:0] data1,data2;
      event done;
      event next;
      
      task generator();   
       for(i = 0; i<10; i++) begin  
          data1 = $urandom();
          $display("Data Sent : %0d", data1);
         #10;
        
         wait(next.triggered);
         
        end
        
       -> done; 
      endtask
      
      
      
      task receiver();
         forever begin
           #10;
          data2 = data1;
          $display("Data RCVD : %0d",data2);
          ->next; 
        end
       
      endtask
      
      
      
      
      task wait_event();
         wait(done.triggered);
        $display("Completed Sending all Stimulus");
         $finish();
      endtask
      
      
      
     initial begin
        fork
          generator();
          receiver();
          wait_event();
        join 
       
       ///////
         
      end
      
      
    endmodule
```
## Demonstration of FORK_JOIN
```
   module tb;
      
          task first();
            $display("Task 1 Started at %0t",$time);
          #20;      
            $display("Task 1 Completed at %0t",$time);     
        endtask
      
      
        
        task second();
          $display("Task 2 Started at %0t",$time);
          #30;      
         $display("Task 2 Completed at %0t",$time);     
        endtask
      
      
      
        task third();
          $display("Reached next to Join at %0t",$time);     
        endtask
      
      
      initial begin
        
     
        
        
        fork
          
          first();
          second();
          
        join
        
        
          third();
     
        
      end
      
      
      
    endmodule
```

## Understanding FORK_JOIN_ANY
```
   module tb;
      
          task first();
            $display("Task 1 Started at %0t",$time);
          #20;      
            $display("Task 1 Completed at %0t",$time);     
        endtask
      
      
        
        task second();
          $display("Task 2 Started at %0t",$time);
          #30;      
         $display("Task 2 Completed at %0t",$time);     
        endtask
      
      
      
        task third();
          $display("Reached next to Join at %0t",$time);     
        endtask
      
      
      initial begin
        
     
        
        
        fork
          
          first();
          second();
          
        join_any
        
        
          third();
     
        
      end
      
      
      
    endmodule

```

## Understanding FORK JOIN_NONE
```
   module tb;
      
          task first();
            $display("Task 1 Started at %0t",$time);
          #20;      
            $display("Task 1 Completed at %0t",$time);     
        endtask
      
      
        
        task second();
          $display("Task 2 Started at %0t",$time);
          #30;      
         $display("Task 2 Completed at %0t",$time);     
        endtask
      
      
      
        task third();
          $display("Reached next to Join at %0t",$time);     
        endtask
      
      
      initial begin
        
     
        
        
        fork
          
          first();
          second();
          
        join_none
        
        
          third();
     
        
      end
      
      
      
    endmodule
```

## Usage of FORK JOIN in Testbench
![[Pasted image 20241116174734.png]]
```
task pre_test();
     driv.reset();
endtask

task test();
 fork 
 gen.main();
 driv.main();
 mon.main();
 sco.main();
 join_any
 endtask

task post_test();
  wait(done.triggered);
  wait(gen.count == driv.trans);
  wait(gen.count == sco.trans);
  endtask

///runtask
task run();
  pre_test();
  test();
  post_test();
  $finish;
  endtask
  endclass
  
  ```
## Understanding Semaphore

```
class first;

 rand int data; 
 constraint data_c (data < 10; data > 0;)

endclass

class second; 
rand int data;
constraint data_c (data > 10; data < 20;)
endclass

class main; 
semaphore sem; 
first f; 
second s; 

int data; 
int i = 0; 

task send_first();
     sem.get(1);
     
for (i = 0; i<10; i++) begin
   f.randomize();
   data = f.data;
   $display("First access Semaphore and Data sent : %0d", f.data);
   #10;
   end
   sem.put(1);
   $display("Semaphore Unoccupied");
   endtask

task send_second();
  sem.get(1);
  for(i = 0; i<10; i++) begin
  s.randomize();
  data = s.data;
  $display("Second access Semaphore and Data sent : %0d", s.data)
  #10;
  end
  sem.put(1);
  endtask

task run();
 sem = new(1);
 f = new();
 s = new();

fork
send_first();

send_second();
join

endtask
endclass

module tb;

main m;

initial begin
m = new();
m.run();
end

initial begin
#250;
$finish();
end

endmodule
```

## Understanding Mailbox
