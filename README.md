Download Link: https://assignmentchef.com/product/solved-hpc-project-22-memory-hierarchies-and-matrix-matrix-multiplications
<br>



<h1>1.      Explaining the impact of memory hierarchies [25 points]</h1>

Data can be stored in a computer system in many different ways. CPUs have a set of registers, which can be accessed without delay. In addition, there are one or more small but very fast caches holding copies of recently used data items. The main memory is much slower, but also much larger than cache. This is typically a complex hierarchy, and it is vital to understand how data transfer works between the different levels in order to identify performance bottlenecks. Caches are low-capacity, high-speed memories that are commonly integrated on the CPU die. The need for caches can be easily understood by realizing that data transfer rates to main memory are painfully slow compared to the CPU’s arithmetic performance. Figure 1: Memory hierarchy of a multicore arCaches can alleviate the effects of the DRAM gap in many cases. <sup>chitecture.</sup>

Usually there are several levels of cache (see Figure 1), called L1D

(D stands for data, L1 is usually shared with instruction cache), L2, and L3 respectively. When the arithmetic unit (AU) executes an instruction (e.g. add, mult) it assumes that the operands are located in the registers. If they are not, the CPU first needs to issue load instructions to fetch the data from some location in the memory hierarchy. Whenever the CPU issues a load request for transferring a data item to a register, first-level cache logic checks whether this item already resides in cache. If it does, this is called a cache hit and the request can be satisfied immediately, with low latency. In case of a cache miss, however, data must be fetched from outer cache levels or, in the worst case, from the main memory.

Caches can only have a positive effect on performance if the data access pattern of an application shows some locality of reference. More specifically, data items that have been loaded into a cache are to be used again “soon enough” to not have been evicted in the meantime. This is also called temporal locality. We will exploit temporal locality to improve performance of the code in part 2 of this mini-project. In this part we will benchmark the memory subsystem to see the effect of the memory hierarchy. A detailed explanation of memory hierarchy can be found in <a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a>.

<h2>Problem statement</h2>

<ol>

 <li>Identify the parameters of the memory hierarchy on the compute node of the ICS cluster:</li>

</ol>

<table width="263">

 <tbody>

  <tr>

   <td width="160">Main memory</td>

   <td width="72"><em>…</em></td>

   <td width="31">GB</td>

  </tr>

  <tr>

   <td width="160">L3 cache</td>

   <td width="72"><em>…</em></td>

   <td width="31">MB</td>

  </tr>

  <tr>

   <td width="160">L2 cache</td>

   <td width="72"><em>…</em></td>

   <td width="31">kB</td>

  </tr>

  <tr>

   <td width="160">L1 cache</td>

   <td width="72"><em>…</em></td>

   <td width="31">kB</td>

  </tr>

 </tbody>

</table>

1

You might find the following useful:

$ module load likwid

$ likwid-topology

$ cat /proc/meminfo

<ol start="2">

 <li>The directory membench on GitHub and the iCorsi course webpage contains

  <ul>

   <li>c – a program in C to measure the performance (benchmark) of different memory access patterns;</li>

   <li>Makefile – a Makefile to compile and run the code;</li>

   <li>gnuplot – a GnuPlot script for displaying performance results;</li>

   <li>sh – a bash script for collecting performance results;</li>

   <li>sh – a bash script for displaying performance results;</li>

  </ul></li>

</ol>

Compile membench.c into membench binary and run it using the provided Makefile:

<ul>

 <li>on your local machine, e.g. laptop (you may need to install gnuplot):</li>

</ul>

$ cd membench

$ make

$ ./run_membench.sh

<ul>

 <li>on the ICS cluster : – compile on login node</li>

</ul>

$ cd membench

$ module load gcc gnuplot

$ make

– start batch job from login node

$ sbatch run_membench.sh

(In the case of the ICS cluster , the resulting generic.ps will be available in few minutes (check the job status with squeue).)

<ol start="3">

 <li>Using the resulting generic.ps files (view them with your favorite PDF viewer) and membench.c program source, characterize the memory access pattern used in the following cases:

  <ul>

   <li><em>csize </em>= 128 and <em>stride </em>= 1;</li>

   <li><em>csize </em>= 2<sup>20 </sup>and <em>stride </em>= <em>csize/</em>2.</li>

  </ul></li>

 <li>Analyze the resulting generic.ps file produced by membench on the ICS cluster (open generic.ps file with your favorite PDF viewer):

  <ul>

   <li>Which array sizes and which stride values demonstrate good temporal locality? Please explain.</li>

  </ul></li>

</ol>

Please include the answers in your Latex report. It should also contain the generic.ps files produced by membench on the ICS cluster and on your local machine (please specify the type and operating system of the local machine you used) and an explanation of the resulting graph in detail.

<h1>2.      Optimize Square Matrix-Matrix Multiplication [60 points]</h1>

<h2>Problem statement</h2>

Your second task in this project<a href="#_ftn2" name="_ftnref2"><sup>[2]</sup></a> is to write an optimized matrix multiplication function on the ICS computer. We will give you a generic matrix multiplication code (also called matmul or dgemm), and it will be your job to tune our code to run efficiently on the ICS processors.

Write an optimized single-threaded matrix multiply kernel. This will run on only one core.

<h3>Matrix multiplication</h3>

Matrix multiplication is the basic building block in many scientific computations; since it is an O(<em>n</em><sup>3</sup>) algorithm, these codes often spend a lot of their time in matrix multiplication. However, the arithmetic complexity is not the limiting factor on modern architectures. The actual performance of the algorithm is also influenced by the memory transfers. We will illustrate the effect with a common technique for improving cache performance, called blocking. Please refer to the additional material on the course webpage, titled <em>Motivation for Improving Matrix Multiplication </em>or in the book. Since we want to write fast programs, we must take the architecture into account. The most naive code to multiply matrices is short, simple, and very slow:

<table width="675">

 <tbody>

  <tr>

   <td width="675"><strong>for </strong>i = 1 <strong>to </strong>n<strong>for </strong>j = 1 <strong>to </strong>n<strong>for </strong>k = 1 <strong>to </strong>nC[i,j] = C[i,j] + A[i,k] * B[k,j] <strong>end</strong><strong>end end</strong></td>

  </tr>

 </tbody>

</table>

Instead, we want to implement the algorithm that is aware of the memory hierarchy and tries to minimize the number of references to the slow memory (please refer to the “Motivation for Improving Matrix Multiplication” document provided on the HPC course web page mini-project-info.pdf for more detailed explanation):

<table width="675">

 <tbody>

  <tr>

   <td width="675"><strong>for </strong>i=1 <strong>to </strong>n/s <strong>for </strong>j=1 <strong>to </strong>n/sLoad C_<em>{i,j} </em>into fast memory <strong>for </strong>k=1 <strong>to </strong>n/sLoad A_<em>{i,k} </em>into fast memoryLoad B_<em>{k,j} </em>into fast memoryNaiveMM (A_<em>{i,k}</em>, B_<em>{k,j}</em>, C_<em>{i,j}</em>) using only fast memory <strong>end for</strong>Store C_<em>{i,j} </em>into slow memory <strong>end for end for</strong></td>

  </tr>

 </tbody>

</table>

<h3>Starter Code</h3>

Download the starter code: Directory matmul

The directory contains starter code for the matrix multiply. It contain the following source files:

<ul>

 <li>dgemm-blocked.c – A simple blocked implementation of matrix multiply. It is your job to optimize the square dgemm() function in this file.</li>

 <li>dgemm-blas.c – A wrapper which calls the vendor’s optimized BLAS implementation of matrix multiply (here, MKL).</li>

 <li>dgemm-naive.c – For illustrative purposes, a naive implementation of matrix multiply using three nested loops.</li>

 <li>c – A driver program that runs your code. You will not modify this file, except perhaps to change the MAX SPEED constant if you wish to test on another computer (more about this below).</li>

 <li>Makefile – A simple makefile to build the executables.</li>

 <li>sh – Script that executes all three executables and produces log files (*.data) that contain the performance logs.</li>

 <li>sh – Script that plots the data in the performance logs and produces a figure showing the results.</li>

</ul>

<h3>Running our Code</h3>

The starter code should work out of the box. To get started, we recommend you to log into the ICS cluster and download the first part of the assignment. This will look something like the following:

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="0a7f796f784a66656d6364">[email protected]</a>]$ git pull

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="ef9a9c8a9daf8380888681">[email protected]</a>]$ cd matmul

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="7104021403311d1e16181f">[email protected]</a>]$ ls Makefile benchmark.c dgemm-blas.c dgemm-blocked.c dgemm-naive.c run_matrixmult.sh

Next let’s build the code.

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="89fcfaecfbc9ecfca4e5e6eee0e7">[email protected]</a>]$ module load gcc intel-mkl

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="b4c1c7d1c6f4d1c199d8dbd3ddda">[email protected]</a>]$ make

<table>

 <tbody>

  <tr>

   <td width="62"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

We now have three binaries: benchmark-blas, benchmark-blocked, and benchmark-naive. The easiest way to run the code is to submit a batch job. We have already provided batch files which will launch jobs for each matrix multiply version using one core:

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="7603051304361a19111f18">[email protected]</a>]$ sbatch run_matrixmult.sh

Our jobs are now submitted to the ICS cluster ’s job queue. We can now check on the status of our submitted jobs using a few different commands.

When our job is finished, we’ll find new files in our directory containing the output of our program. For example, we will find the files matrixmult-xxx.out and matrixmult-xxx.err . The first file contains the standard output of our program, and the second file contains the standard error. Additionally, the performance data are stored in *.data files. The script also generates timing.ps file, a visual representation of the results. You can copy the timing.ps file to your laptop and open it with your favorite PDF file viewer.

<h3>Interactive Session</h3>

You may find it useful to launch an interactive session when developing your code. This lets you compile and run code interactively on a compute node that you have reserved. In addition, running interactively lets you use the special interactive queue, which means you’ll receive you allocation quicker. The benchmark.c file generates matrices of a number of different sizes and benchmarks the performance. It outputs the performance in FLOPS and in a percentage of theoretical peak attained. Your job is to get your matrix multiply’s performance as close to the theoretical peak as possible.

<h3>Theoretical Peak</h3>

Our benchmark reports numbers as a percentage of theoretical peak. Here, we show you how we calculate the theoretical peak of the ICS cluster ’s Haswell processors. If you’d like to run the assignment on your own processor, you should follow this process to arrive at the theoretical peak of your own machine, and then replace the MAX SPEED constant in benchmark.c with the theoretical peak of your machine.

<h3>One Core on the ICS cluster</h3>

One core has a clock rate of 2.30 GHz, so it can issue 2.3 billion instructions per second. Haswell processors also have a 256-bit vector width, meaning each instruction can operate on 8 32-bit data elements at a time. Furthermore, the Haswell microarchitecture includes a fused multiply-add (FMA) instruction, which means 2 floating point operations can be performed in a single instruction. So, the theoretical peak of the ICS cluster ’s Haswell node is

<ul>

 <li>3 GHz* 8-element vector * 2 ops in an FMA = 36.8 GFlops/s</li>

</ul>

Note that the matrices are stored in C style row-major order. However, the BLAS library expects matrices stored in column-major order. When we provide a matrix stored in row-wise ordering to the BLAS, the library will interpret it as its transpose. Knowing this, we can use an identity <em>B<sup>T</sup>A<sup>T </sup></em>= (<em>AB</em>)<em><sup>T </sup></em>and provide matrices <em>A </em>and <em>B </em>to BLAS in rowwise storage, swap the order when calling dgemm and expect the transpose of the result, (<em>AB</em>)<em><sup>T</sup></em>. But the result is returned again in column-wise storage, so if we interpret it in rowwise storage, we obtain the desired result <em>AB</em>. Have a look at dgemm-blas.c to see how the <em>A </em>and <em>B </em>are passed to dgemm. Also, your program will actually be doing a multiply and add operation <em>C </em>:= <em>C </em>+ <em>A </em>· <em>B</em>. Look at the code in dgemm-naive.c or study the dgemm signature if you find this confusing. The driver program supports result validation (enabled by default). So during the run of benchmark-blocked binary compiled from the squaredgemm code you wrote, the result correctness will be automatically checked for different matrix sizes.

<strong>2.1. Optimizing the matrix multiplication </strong>Now, it’s time to optimize!

<ul>

 <li>The dgemm-blocked.c contains the naive implementation of the square matrix multiply. Modify the code so that it performs blocking. Test your code and tune block sizes to obtain the best performance.</li>

 <li>Compare performance of your implementation to the Intel MKL by compiling and running the driver program and visualizing the performance results.</li>

</ul>

<ol start="3">

 <li><strong> Quality of the Report [15 Points]</strong></li>

</ol>

Each project will have 100 points (out of 15 point will be given to the general written quality of the report).


