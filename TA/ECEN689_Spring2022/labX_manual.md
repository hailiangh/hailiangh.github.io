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
[!fig1](./pics/)  

### 1.1.2 Iterative Implementation
As shown in the below figure, for an iterative approach, instead of implementing "n" iterations of the algorithm, one iteration is implemented. This implemented design is used "n" clock cycles to achieve the final output.  
[!fig2](.pics/)  
Figure 1 and 2 are taken from:
[https://link.springer.com/content/pdf/10.1007%2F978-0-387-36682-1_9.pdf](https://link.springer.com/content/pdf/10.1007%2F978-0-387-36682-1_9.pdf).  

## Using Virtual Input Output (VIO) to Drive Input and Read Output of the AES  
Unlike other labs, in this lab, you will be using a **virtual input output (VIO)** IP core to drive input data and read the output data from the AES. We will not be implementing the processor core in this lab.  
As shown in the below figure, th top-level design has three major sections:
- **Phase-locked loop:** This will take the input clock from the FPGA board (125MHz) and generates a 200MHz output clock. The **pll lock** signal from the PLL indicates if the output clock of the PLL is stable. For example, if pll lock = 1, the output of PLL is stable. Otherwise, there is no output clock generated. Please refer to the lab 2 manual for detailed instructions on instantiating PLL in the design.  
- **AES:** Undergraduate students are required to implement only the pipelined version of the AES. Graduate students are required to implement both the pipelined and iterative versions of the AES expalined in Section 1.1.  
- **VIO IP core:** As shown in the figure below, the VIO drives the plaintext and key to the AES and receives the ciphertext from the AES. The following are the connections to the VIO:  
  - The clock signal connecting the **clk** input of the VIO should be the same clock driving the AES. This ensures that the signals to and from the VIO are synchronous to the signals in the AES.  
  - The inputs and outputs of the VIO are called **probe_in** and **probe_out**, respectively. As illustrated in the figure below, the **probe_in** signals receive the ciphertext and its valid signal from the AES. Likewise, the **probe_out** signals drive the plaintext, key, and their corresponding valid signals.  
As we already have information on PLL instantiation and AES design, we will now focus on instantiating the VIO IP core required for accessing the AES design during runtime.  




## 4. Pre-lab Submission
Pre-lab is not required for this lab since this is a two-week lab. A balanced load would be **PE.v** and **array.v** in the first week, and **systolic_array.v** in the second week.  


## 5. Post-lab Submission
- Please only submit one PDF file, containing the following items:    
    - Screenshots of the terminal after running the command `./lab9_cnn_test  
    - A few words explaining the results
    - Screenshots of your code in this design
    - Screenshots of any simulations you have for partial credits
- Please name the PDF file as "Lab#_Postlab_Section#_LastName_FirstName.pdf".  
- Please submit the PDF file on Canvas before April 22 (Friday) 11:59 pm.  


