#Partial Reconfiguration Controller for Xilinx FPGAs
This controller is provided to help implement adaptive systems that leverage partial reconfiguration.
The top file for the application is uapp.v. 
The DDR3 memory controller should be generated by the user.
It is done using Xilinx's Memeory Interface Generator (MIG) application.
Detailed instructions for generating the DDR3 controller for ML605 evaluation board is available at http://www.xilinx.com/support/documentation/boards_and_kits/xtp047.pdf
#File descriptions
##uapp.v
This is the most design file, containg the dma controller, ICAP controller, UART controller, and the clock generator.
##dma_top.v
This is the top file for the DMA controller.
It also contains the statistics block.
##ddr_controller.v
This file contains the module for controlling the DDR read and write operations.
##dma_ctrl.v
This module initiates and controls the DMA operations under the commands issued through the UART controller.
It instantiates the command interpreter for commands issued by the user.
##cmd_int.v
This module decodes the commands issued through the UART and generates necessary control signals.
##icap_controller.v
This module contains the state machine to control the ICAP macro write operations.
It also instantiates the buffer to temporarily store the partial bitstreams before sending to the ICAP macro.
##clocks.v
This module contains the multi-mode clock manager for generating the required ICAP clock frequency .
##uart_top.v
This is the top file for the UART controller.

##User Commands
###CMOD : Command Mode
This is the mode in which the system receives commands from the host system. In this mode the data sent from the hyper terminal is looped backed to the terminal, so that the user can see it. After executing several other commands such as DMOD, SRST, RST1, RST2, PICP, CINT, SCFQ, and NCON, this command should be executed to bring the system to the idle mode before executing any other command.
###SRST : Soft Reset
When this command is executed, all the logic except the DDR memory controller is reset. This command should be executed for clearing the statistics counters before transferring a new bitstream from the memory to the ICAP.
###SLEN : Set Length
This command is used for setting the DMA transfer length. This might be needed for transferring partial bitstreams to the system memory as well as to transfer partial bitstreams from the memory to the ICAP controller. The length is specified in number of bytes and it should be a 6 digit number. 
###SADR : Set Address
This command is used for setting the system memory addresses for DMA read and write operations. The address should be decimal and it should be a 6 digit number.
###DMOD: Data Mode
This command is used to transfer partial bitstreams from the host system to the on board memory. Both SLEN and SADR commands should be executed before this command for arming the DMA controller to receive data from the host system. In DMOD, the data sent from the terminal is not looped back. CMOD command should be executed at the end of data transfer.
###RST1: Read Statistics 1
This command reads the contents of the first statistics register. This register gives the overall system performance. The result is shown in binary.
###RST2: Read Statistics 2
This command reads the contents of the second statistics register. This register gives the ICAP controller performance. The result is shown in binary.


###CINT : Configure Internally
Command to configure the system using the information from the configuration pointer buffer. A decimal number should follow CINT, which indicates the ordinal number of the partial bitstream. 
###SCFQ : Set Clock Frequency
Command used to change the ICAP controller clock frequency. Reissuing this command will revert back the clock frequency to 100MHz.
###NCON : Number of Configurations
Command used to find out how many times each partial bitstream is used for configuration using the configuration pointer buffer. 
###Example:
For transferring 1000 bytes partial bitstream and 2000 bytes partial bitstream to the system memory and then use for configuration  
SRST                      //Soft reset the system  
CMOD                      //Enter command mode  
SLEN 001000               //Set the transfer length and DDR address  
SADR 000000  
DMOD                      //Now transfer the bitstream  
CMOD                      //Enter command mode  
SLEN 002000               //Set the transfer length and DDR address for second partial bitstream  
SADR 100000  
DMOD                      //Transfer the second bitstream  
CMOD                      //Enter the command mode  
SLEN 001000               //Now for configuring the first bitstream, again arm the DMA controller  
SADR 000000  
PICP                      //Configure using the DMA engine and ICAP controller  
CMOD                      //Enter command mode  
RST1                      //Read the statistics registers  
CMOD  
RST2  
CMOD  
SRST                      //Clear the statistics registers  
CMOD                      //Enter the command mode  
CINT1                     //Configure the system with the second bitstream using the configuration pointer buffer  

