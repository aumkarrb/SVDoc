- Verification Plan
- Directed Test Vs Constraint Random Test
- Layered Testbench Architecture
- Individual Components of Layered Architecture

## Understanding Verification Plan
![[Pasted image 20241110103739.png]]

| Sr no. | Testcase | Description                                         | Feature Covered                                                                                          |
| ------ | -------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 1.     | rst_high | Default values for dout when reset is asserted      | This verify the behaviour of the dout at the start of operation when reset is high for first few cycles. |
| 2.     | rst      | reset is asserted during read and write transaction | Dout must hold its initial value whenever reset is asserted in the middle of ongoing transaction         |
| 3.     | wr       | wr is asserted after reset is deasserted            | Verify Valid data store in the memory during Write transaction                                           |
| 4.     | wr       | wr is asserted when rst is asserted                 | Data should not be added to the Memory                                                                   |
| 5.     | wr_low   | wr is deasserted when reset is deasserted           | dout must return valid value store during previous write transaction                                     |
| 6.     | wr_low   | wr is deasserted when reset is asserted             | dout must stay at zero                                                                                   |
| 7.     | wr_low   | wr is deasserted when rst is deasserted             | reading data from address without valid data, dout must return zero                                      |

| Testcase Name      | Summary                                                   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Weight | Goal                                                                         | Covergroup        | Specification                                 |
| ------------------ | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------- | ----------------- | --------------------------------------------- |
| apb_slv_error_test | Test error response functionality                         | Generate address that are outside of Address space of DUT.     Generate write and read operation with wrong address and poll for slave err response signal (PSLVERR)to go high. The test passes if DUT can detect the Address out of the bound error and set PSLVERR signal to high.                      Run this testcase with APB_SLV_WAIT_FUNC_EN = 1 user define. The min and max wait delays can be defined using APB_SLV_MIN_WAIT_CYC and APB_SLV_MA_WAIT_CYC user defines. | 1      | op_type == READ, WRITE  Address = OUT OF BOUND ADDRESS data = patterns       | op_type ADDR DATA | APB_Slave_Core_SRAM_Design_Specification.docx |
| apb_slv_arr_test   | Test error response functionality with wait state enabled | Generate address that are outside of Address space of DUT.      Generate write and read operation with wrong address and poll for slave err response signal(PSLVERR) to go high.                           The test passes if DUT can detect the Address out of bound error and set PSLVERR signal to high.                                                                                                                                                                        | 1      | op_type == READ, WRITE       Address =OUT OF BOUND  ADDRESS  data = patterns | op_type ADDR DATA | APB_Slave_Core_SRAM_Design_Specification.doc  |
|                    |                                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |        |                                                                              |                   |                                               |

## Directed Test Vs Constraint Random Test
![[Pasted image 20241110133038.png]]![[Pasted image 20241110133518.png]]
![[Pasted image 20241110134222.png]]
![[Pasted image 20241110134424.png]]

## Layered Architecture
![[Pasted image 20241110141057.png]]
![[Pasted image 20241110141658.png]]

 # Individual Components of TB
![[Pasted image 20241110142541.png]]
 1. Transaction: Contain variables fore all the inputs/outputs port present in DUT to share among                              classes.
 2. Generator: Generate random stimulus and send it to Driver using IPC
 3. Driver: Receive Stimulus from Generator and Trigger respective Signals of DUT with help of INTERFACE
 4. Monitor: Receive Response from DUT and send it to scoreboard using IPC
 5. Scoreboard: Compare response of DUT with Expected / Golden Data
 6. Environment: Hold GEN,DRV,MON,SCO together.
 7. Test: Hold Env and Control simulation process.