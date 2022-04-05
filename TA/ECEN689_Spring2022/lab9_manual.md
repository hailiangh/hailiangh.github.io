---
layout: post
title: Lab 9 - Systolic Array Based CNN Design on FPGAs
description: ECEN 489/689 - Spring 2022
use_math: true
use_code: true
---
## 1. Introduction
In this lab we need to design systolic arrays for convolutional neural networks (CNN).  

### 1.1 Structure of systolic arrays
The structure discussed here is similar to the one described by authors *Xuechao Wei, Cody Hao Yu, Peng Zhang, Youxiang Chen, Yuxin Wang, Han Hu, Yun Liang, and J. Cong* in the paper, **“Automated Systolic Array Architecture Synthesis for High Throughput CNN Inference on FPGAs”** published in 2017 at the  *54th ACM/EDAC/IEEE Design Automation Conference (DAC), pp. 1–6*.  
![fig1](./pics/lab8_manual_SystolicArray_3D.png)  
In the , the terms **WB**,**IB**,**OB** refer to the weight buffers, input buffers and output buffers. PE (Processing Element) is the basic unit similar to Lab8. Its structure in this lab is shown in the figure below.  
![fig2](./pics/lab8_manual_SystolicArray_2D.png)  

### 1.2 Convolution calculation
For each layer of the CNN, we need to do the convolution. The pseudo-code is as follows:
```
for (o = 0; o < do; o++) {
  for (i = 0; i < di; i++) {
    for (c = 0; c <= dc - dkc - sc; c += sc) {
      for (r = 0; r <= dr - dkr - sr; r += sr) {
        for (kr = 0; kr < dkr; kr++) {
          for (kc = 0; kc < dkc; kc++) {
            OUT[o][r/sr][c/sc] += W[o][i][kr][kc] * IN[i][r+kr][c+kc]
          }
        }
      }
    }
  }
}
```
where *do, di, dc, dr, dkr, dkc* are the maximum size limits in each direction. The strides of the kernel in the directions of row and column are denoted by *sr* and *sc* respectively. *W* represents the weights, *IN* represents the data of the input image and *OUT* denotes the output. The convolution is illustrated in the figure below.  
![fig3](./pics/lab8_manual_SystolicArray_2D.png) 
For each PE, it can do multiplication and accumulate the data. So we need to input all the data required for each `OUT[o][r/sr][c/sc]` into one PE. For example, assume `sc = sr = 1`, to calculate `OUT[1][2][3]`, we need to input the pairs `(W[1][0][0][0], IN[0][2][3]), (W[1][1][0][0], IN[1][2][3]),… ……(W[1][0][1][0], IN[0][2+1][3]), (W[1][1][1][0], IN[1][2+1][3]), (W[1][2][1][0], IN[2][2+1][3])...` to one PE and finally let the the PE output the data.  

### 1.3 Scheduling of the convolution
Now we need to map the convolution to the systolic array. It is important to choose which PE would calculate which `OUT[o][r/sr][c/sc]`. In the proposed systolic array, each PE will transmit its two inputs (`W` and `IN`) to the next two PEs near it, so `W` and `IN` should also be available for use by the near PE.  
In this lab, we map the calculation as described below:  
We map **“o”** along the row, and **“c”** along the column. This means, the PE at row 1 column 2 (starting from row 0 and column 0) needs to calculate `OUT[1][r][2]`.  
`i`,`kr` and `kc` are regarded as “vectors”, which means that for each PE, `W` and `IN` with different `i`, `kr` and `kc` are sequentially inputted. For the sequential inputs, `i` changes first, then `kr`, and then `kc`.  
If the size of the systolic array is smaller than `do*dc`, then `o` and `c` should be partially mapped to the systolic array. For example, assume `sr = sc = 1`, if `do = 20`, `dc = 10`, and the systolic array is `4 x 5`, then for the first time, `o = 0 to 3` are mapped to row and `c = 0 to 4` are mapped to column. After the computing is complete, `o = 0 to 3` and `c = 5 to 9` are mapped to row and column. Then, `o = 4 to 7` and `c = 0 to 4` are mapped, and so on.  
After all the `o` and `c` are mapped to the systolic array and the calculation is finished, `r` is increased by 1, and all the above steps are repeated until all the `OUT[o][r/sr][c/sc]` are calculated.  

The scheduling of a systolic array of 3 rows and 4 columns are illustrated in the figure below, where `sr = sc = 1`.  
![fig4](./pics/lab8_manual_SystolicArray_2D.png)  
Note that with rows and columns increasing, the sequence of vectors are inputted with delays to synchonize the input of `W` and `IN` to one PE.  
In this way, `W` and `IN` inputted to one PE can also be transmitted and used by PEs near it. For example, after `W[1][0][0][0]` and `IN[0][0][2]` arrives at PE at row 1 and column 2 at the same clock cycle. `W[1][0][0][0]` can also be transmitted to the PE at row 1 and column 3 at the next clock cycle, together with `IN[0][0][3]` which is needed to calculate `OUT[1][0][3]`.  

### 1.4 Proposed CNN
The figure below shows the structure of the CNN we use. We will use it to do the inference on [MNIST](https://en.wikipedia.org/wiki/MNIST_database) dataset.  
![fig5](./pics/lab8_manual_SystolicArray_2D.png)  

For simplicity, we ignore the pooling layer. The activation function of each layer is **ReLU**, except the last layer. The strides `sr` and `sc` of the two convolution layers are both 2.  

Although the last two layers are fully connected layers, it can also be regarded as convolution layer for calculation purpose. If the input neurons are `m`, and output neurons are `n`, then, by setting `dkr = dkc = dr = dc = 1`, and `di = m`, `do = n`, we can reuse our convolution layer’s systolic array structure or scheduling approach to calculate fully connected layers, as shown in the figure below.  
![fig6](./pics/lab8_manual_SystolicArray_2D.png)  


## 2. Lab Designs
In this section, we need to implement the systolic array module in Vivado. Before you proceed, please download **"Lab8_student_code.zip"** from Piazza and extract it. After extraction, you will get a folder named as **"Lab8_student_code/"**. Copy the folder **"base_vivado"** and rename it as **"lab8_vivado"**. From the source panel, remove unnecessary source files. Open the project by double-click on **"lab8_vivado/base/base.xpr"**.

In this lab, we will implement the 3-D systolic array design in **"systolicarray_1.v"**, and the 2-D systolic array design in **"systolicarray_2.v"**. Please use **8-bit signed fixed-point** number for calculation, with **4 bits** for the fractional part.  

In this lab, all the matrices in the sequential form are arranged column by column, and then row by row. For example, if a matrix $$m$$ has 3 rows and 2 columns, we will have an array to save this matrix as $$\lbrack m_{32}, m_{31}, m_{22}, m_{21}, m_{12}, m_{11} \rbrack$$ ,where $$m_{ij}$$ refers to data at row $$i$$ and column $$j$$ of the matrix $$m$$.  

- In **"systolicarray_1.v"**,$$mi0$$, $$mi1$$ are two inputs representing the two matrices. The output matrix is $$mor$$, resulting from the operation $$mi0 \times mi1$$. You are required to design the combinational logic between $$mi0$$, $$mi1$$ and $$mo$$, where $$mo$$ is the output result from operation $$mi0 \times mi1$$ of the combinational logic.  
- Please write a testbench **"systolic_array_tb.v"** to calculate the  
  $$
  \begin{bmatrix} 0.5 & 1 \\ 1 & 0.5\end{bmatrix} \times \begin{bmatrix} 1 & 2 \\ 3 & 4\end{bmatrix}
  $$  
  Run the behavioral simulation and take a screenshot of the result. Please include it in the pre-lab submission with explanation on the result.

- For the 2-D design of the systolic array, please implement it in **"systolicarray_2.v"**. Please pay attention to stream in the entries of the two input matrices in the correct order. After the design, please use the same test bench file to do the same matrix calculation, and include the screenshot in your post-lab submission.  

## 3. Implementation on the FPGA
In this section, we will implement the design on the FPGA.  
- Right click **"top_sysarr.v"** in the **"source"** panel and click **"set as top"** (If this file is shown in bold font, it is already the top module).   

- The block RAM settings for this lab is shown in the table below.
  
    | Block RAM Name | Memory Type |    Port A Settings | Port B Settings|
    | ------------- | ------------- | ------------- | ------------- |
    | blk_mem_gen_0  | True Dual Port  | Width: 8 Depth: 65536 Read First Always Enabled  | Same as Port A  |  

- Click on **"Generate Bitstream"** to invoke the design flow and generate the bitstream. After the bitstream is generated, click **"File -> Export -> Export Hardware"**. Check the box **"Include Bitstream"**, click **"OK"**.
- Please launch SDK and generate the boot image (**BOOT.bin**) as in the previous lab with one exception:  
    Use the bitstream file **base/base.sdk/top_viterbi_hw_platform_0/top_systolicarray.bit**.
- Copy the updated **BOOT.bin** and **lab8_sysarr_test** into your SD card, boot the FPGA and run the test with command:
    ```  
    ./lab8_sysarr_test  
    ```  
- Take a screen shot of the terminal when the result shows.
- Unmount the SD card, exit the serial communication and turn off your FPGA.

- Some commonly used commands:  
    ```
    picocom -b 115200 -r -l /dev/ttyUSB1
    mount /dev/mmcblk0p1 /mnt/
    cd /mnt/
    insmod transfpga.ko
    mknod /dev/transfpga c 245 0
    ./lab8_sysarr_test 
    cd /
    umount /mnt/
    ```

## 4. Pre-lab Submission
- Please only submit one PDF file, containing your code and simulations for the 3-D systolic array design.   
- Please name the PDF file as "Lab#_Prelab_Section#_LastName_FirstName.pdf".  
- Please submit the PDF file on Canvas before April 4 (Monday) 11:59 pm.  


## 5. Post-lab Submission
- Please only submit one PDF file, containing the following items:  
    - Screenshots of the behavioral simulation of the 2-D systolic array design  
    - Screenshots of the terminal after running the command `./lab8_sysarr_test  
    - A few words explaining the results
    - Screenshots of your code in this design
- Please name the PDF file as "Lab#_Postlab_Section#_LastName_FirstName.pdf".  
- Please submit the PDF file on Canvas before April 8 (Friday) 11:59 pm.  

