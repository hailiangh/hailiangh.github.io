---
layout: post
title: Lab 10 - Implementation of Advanced Encryption Standard Algorithm
description: ECEN 489/689 - Spring 2022
use_math: true
use_code: true
---
## 1. Introduction
In this lab, you will be implementing the iterative and pipelined versions of the Advanced Encryption Standard cryptographic algorithm. You will also compare and contrast: power, area, throughput, and latency incurred by these two architecture variants.    
For this lab, you are expected to complete the following tasks:
- Implement AES based on iterative architecture.  
- Implement AES based on pipelined architecture.  
- Compare and contrast these two implementations based on power, area, throughput, and latency.  
You have to implement only the encryption module of the AES for both variants: iterative and pipelined.  


### 1.1 About Advanced Encryption Standard
AES is a Federal Information Processing Standards Publications (FIPS) approved cryptographic algorithm used for protecting digital data. The AES performs two functions: encryption and decryption. Encryption converts data to an unintelligible form called ciphertext, whereas decryption converts the data back into its original form, called plaintext. This algorithm has two inputs: the secret key and input data. The input data can be plaintext or ciphertext. The output of the AES will be ciphertext or plaintext, depending on the input. This algorithm falls under the symmetric block ciphers, where the same key is used for encryption and decryption.  

The integral parts of the AES algorithms are the key expansion module and round operation. The AES has ten rounds of input text processing using the round operation. Depending on the power and area requirements, the designer can implement one round operator and use it ten times (iterative implementation) or implement ten round operators. Each operator is used just once (pipelined implementation). For more information on the AES internal design, please refer to Section 5.1 and Section 5.2 in [FIPS AES documentation](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf), Page 13—20.  

Depending on the power and area utilization requirements, the AES algorithm can be implemented based on different architectures such as iterative, pipelined, and using a single SBOX.  In this lab, we will focus on two of these architectures --- iterative and pipelined. 

### 1.1.1 Pipelined Implementation
For a pipeline architecture, all AES rounds are unrolled. That is achieved by repeating one AES round 11 times, as shown in the below figure.  
[!fig1](./pics/labX_manual_fig1.png)  

### 1.1.2 Iterative Implementation
As shown in the below figure, for an iterative approach, instead of implementing "n" iterations of the algorithm, one iteration is implemented. This implemented design is used "n" clock cycles to achieve the final output.  
[!fig2](.pics/labX_manual_fig2.png)  
Figure 1 and 2 are taken from:
[https://link.springer.com/content/pdf/10.1007%2F978-0-387-36682-1_9.pdf](https://link.springer.com/content/pdf/10.1007%2F978-0-387-36682-1_9.pdf).  

## 2. Using Virtual Input Output (VIO) to Drive Input and Read Output of the AES  
Unlike other labs, in this lab, you will be using a **virtual input output (VIO)** IP core to drive input data and read the output data from the AES. We will not be implementing the processor core in this lab.  
As shown in the below figure, th top-level design has three major sections:
- **Phase-locked loop:** This will take the input clock from the FPGA board (125MHz) and generates a 200MHz output clock. The **pll lock** signal from the PLL indicates if the output clock of the PLL is stable. For example, if pll lock = 1, the output of PLL is stable. Otherwise, there is no output clock generated. Please refer to the lab 2 manual for detailed instructions on instantiating PLL in the design.  
- **AES:** Undergraduate students are required to implement only the pipelined version of the AES. Graduate students are required to implement both the pipelined and iterative versions of the AES expalined in Section 1.1.  
- **VIO IP core:** As shown in the figure below, the VIO drives the plaintext and key to the AES and receives the ciphertext from the AES. The following are the connections to the VIO:  
  - The clock signal connecting the **clk** input of the VIO should be the same clock driving the AES. This ensures that the signals to and from the VIO are synchronous to the signals in the AES.  
  - The inputs and outputs of the VIO are called **probe_in** and **probe_out**, respectively. As illustrated in the figure below, the **probe_in** signals receive the ciphertext and its valid signal from the AES. Likewise, the **probe_out** signals drive the plaintext, key, and their corresponding valid signals.  
[!fig3](./pics/labX_manual_fig3.png)  
As we already have information on PLL instantiation and AES design, we will now focus on instantiating the VIO IP core required for accessing the AES design during runtime.  

### 2.1 Instantiating a VIO IP Core
The steps below shows how to instantiate a VIO IP core in the design.  
- Click on the IP Catalog and then search for "VIO," as shown below.  
[!fig4](./pics/labX_manual_fig4.png)  
[!fig5](./pics/labX_manual_fig5.png)  
- Mention the number of input and output probes in the "General option" tab.  
[!fig6](./pics/labX_manual_fig6.png)  
- Provide the port width for each input in the "PROBE_IN ports" tab  
[!fig7](./pics/labX_manual_fig7.png)  
- Provide the port width for each output in the "PROBE_OUT ports" tab. You can also specify the power-on value of these outputs by updating the tabs under "Initial Value (in hex)".  
[!fig8](./pics/labX_manual_fig8.png)
-  Clicking "ok" will open up the following window. Next, click on "Generate" to generate the Verilog and project files corresponding to the configured VIO IO core.    
[!fig9](./pics/labX_manual_fig9.png)
- Instantiate the generated VIO in the top-level file along with the AES and PLL module, as shown below. The following is an example code having the PLL, AES, and VIO instantiations. Please go through all the comments given in the below Verilog file to ensure the design works as intended. 
[!fig10](./pics/labX_manual_fig10.png)

## 3. Testing the AES design using Vivado Simulations
Please use the following testbench template to verify the AES design. The testbench includes a locally generated clock and the AES instance. However, it does not include the PLL and VIO IP cores, as shown below. Depending on the port names of the AES design, the below testbench should be modified. 
[!fig11](./pics/labX_manual_fig11.png)

## 4. Testing the AES design on the FPGA using the VIO IP Core
In this section, we will run the AES design on the FPGA using the VIO IP Core. 
- Generate the bitstream *.bit file consisting of the AES along with the PLL and VIO IP core.  
- If you have the Vivado Design Suite 2018.3 installed in your local machine, please skip this step. Otherwise, download the Vivado Lab Edition 2018.3 software on your local machine from [here](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html). Download the version given under "Vivado Lab Solutions - 2018.3  Full Product Installation" and install the same.  
- Open the Vivado Design Suite (or Lab Edition) 2018.3 software. Click on the "Open Hardware Manager," as indicated in the below figure.  
[!fig12](./pics/labX_manual_fig12.png)  
- Click on the "Open Target", as indicated in the below figure.  

[!fig13](./pics/labX_manual_fig13.png)  
- Click on the "Program Device" to load the bit file into the FPGA.  

[!fig14](./pics/labX_manual_fig14.png)  
- Provide the path where the bitstream (*.bit) and the debug probe file (*.ltx) are stored. The ltx file is required to access the VIO signals onboard. This file is generated along with the *.bit file and both these files can be found in the <project_name>.runs/impl_1 folder.  


[!fig15](./pics/labX_manual_fig15.png)  
- After successful programming, the Vivado will automatically populate the VIO instances in the design, as shown below. Add the VIO input and output signals by clicking on the "+" sign indicated in the red box.   


[!fig16](./pics/labX_manual_fig16.png)  
- You can either add all the signals or choose only the ones required.  

[!fig17](./pics/labX_manual_fig17.png)  

- The tool lists the selected VIO signals. It also mentions if the signal is an input/output.  
[!fig18](./pics/labX_manual_fig18.png)  

- The key and plaintext input values can be modified via the GUI, as shown below. Once the AES encrypts (this will take only few 200 MHz clock cycles), the ciphertext can be read from the dataout signal from the "Value" tab.  
[!fig19](./pics/labX_manual_fig19.png)  


## 5. Resources Required
### 5.1 Understanding the AES internals
- Class notes
- Section 5.1 and 5.2 in the [FIPS AES documentation](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf) (Page 13 - 20)
- [Details on iterative and pipeline architecture](https://link.springer.com/content/pdf/10.1007%2F978-0-387-36682-1_9.pdf).   This document is available via [TAMU Library link](https://tamu.libguides.com/az.php?a=s) -> Choose "SpringerLink" -> Share your TAMU credentials -> Search for "Architectural Designs For the Advanced Encryption Standard". You can download the PDF version of this book for free.  
### 5.2 Understanding VIO
- [User guide](https://www.xilinx.com/support/documentation/ip_documentation/vio/v3_0/pg159-vio.pdf)
- [Video guide](https://www.xilinx.com/video/hardware/logic-debug-in-vivado.html)
### 5.3 Implementing PLL
- Please refer to the lab 2 manual for detailed instructions on instantiating PLL in the design.  


Pre-lab is not required.


## 5. Post-lab Submission
- Please only submit one PDF file, containing the following items:    
    - Screenshots of the terminal after running the command `./lab9_cnn_test  
    - A few words explaining the results
    - Screenshots of your code in this design
    - Screenshots of any simulations you have for partial credits
- Please name the PDF file as "Lab#_Postlab_Section#_LastName_FirstName.pdf".  
- Please submit the PDF file on Canvas before April 22 (Friday) 11:59 pm.  


