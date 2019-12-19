# GPU Computing on ShARC

This lab class will walk you through the process of running GPU code on the University of Sheffield’s new High Performance computing system (ShARC). This process will enable you to develop and execute your GPU code on a machine without a GPU attached. I.e. Should you wish, you will be able to use this process to remotely develop and execute your assignment without access to the High Spec PC lab. ShARC has a number of GPU resources available including two public nodes with 4x Tesla K80 GPUs (8 devices). There are also a number of privately owned GPU nodes including the department of computer sciences DGX-1 with 8x Tesla P100 GPUs.

_Note: Executing code on ShARC requires that we build the code using the NVIDIA compiler `nvcc` in Linux without visual Studio._

## ShARC Accounts

In order to use ShARC you must have an account configured. I have already requested accounts for everyone enrolled int he module and this have been set up for you. You will have recieved an email already requiring you to synchronise your CICS passwords. If you have not already done this then please synchronise your password at [www.shef.ac.uk/cics/password](www.shef.ac.uk/cics/password) by logging on and choosing option 4, synchronise password. There's no need for you to change your password during this process.

## Accessing ShARC GPUS and Initial Setup

The ShARC is accessed through an `ssh` terminal. To login to ShARC from a CICS managed desktop account you can use the PuTTY program (if it is unavailable then you will need to install it via the software centre). Within Putty, set the _“Host Name”_ text field to `sharc.shef.ac.uk`. For some exercises (or your assignment) you may wish to view output images which will require X windows forwarding. To enable this, ensure the _“Enable X Window Forwarding”_ option is selected from the `Connection -> SSH -> X11` tab (see figure below). Select Open and login with your CICS username and password.

{: .center}
![Putty configuration](/assets/images/putty_config.png)

If you are connecting from a machine which is your own (i.e. not a CICS managed desktop) then connect to ShARC using the instructions provided on the [ShARC documentation site](http://docs.hpc.shef.ac.uk/en/latest/sharc/index.html):

[http://docs.hpc.shef.ac.uk/en/latest/hpc/connecting.html](http://docs.hpc.shef.ac.uk/en/latest/hpc/connecting.html).

Once you are logged into a ShARC head node then request an interactive session by typing

    qrshx

This creates an interactive session on a CPU worker node which supports running graphical applications as well as command line programs. The worker node **will not** be a GPU accelerated node and **cannot** be used to execute CUDA applications. The worker node can however be used for compilation (using `nvcc`), interactive editing of GPU code and **submission of jobs to a GPU accelerated worker**. This is a much better solution than running an interactive session on a GPU node as tasks like modifying or building your code do not require exclusive access to a GPU resource.

## Configuring the CPU worker node

To be able to compile CUDA code within your interactive session on the CPU worker node, you will need to add a couple of modules to the environment. Load the latest version of CUDA using the following command:

    module load libs/CUDA

To compile CUDA programs, you also need a compatible version of the GCC compiler suite. Load version 4.9.4 using the following command:

    module load dev/gcc/4.9.4

_Note: Newer versions of GCC are supported on ShARC but may not work correctly with CUDA. Further notes on the modules are available on the ShARC [software documentation site](http://docs.hpc.shef.ac.uk/en/latest/sharc/software/libs/cuda.html#cuda-sharc)._

Test your CUDA module is correctly loaded by calling:

    nvcc --version

You should get the version information similar to the output below.

    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2017 NVIDIA Corporation
    Built on Fri_Nov__3_21:07:56_CDT_2017
    Cuda compilation tools, release 9.1, V9.1.85

## Hello World for GPUs

From your interactive session on the CPU worker node get the _"hello world for GPUs"_ code from Github by cloning the master branch of the `CUDAHelloWorld` repository from the RSE-Sheffield github account. E.g.

    git clone https://github.com/mondus/CUDAHelloWorldCOM4521.git

This will check out the hello world example into the folder `CUDAHelloWorldCOM4521`. Change directory to this folder. e.g.

    cd CUDAHelloWorldCOM4521

Take a look at the contents of this file in the console using `nano` e.g.

    nano helloworld.cu

_Note: If you wish to view/edit this file locally then you can use FPT (e.g. using Filezilla). Files can be transfered to and from your HPC account storage space using the instructions on the [ShARC documentation page](http://docs.hpc.shef.ac.uk/en/latest/hpc/transferring-files.html). MobaXTerm described on the same link can also be used instead of Putty for both SSH connections and remote file editing._

Compile the code using nvcc with the following command (note this is the command line equivalent of the build process in visual studio):

    nvcc helloworld.cu -o helloworld

Better still build your code by setting the specific architecture. E.g. The following will build for both K80 and P100 GPUs (both of which are available on the gputest que). A full list of supported architectures on ShARC is available on the [CUDA section of the ShARC documentation site](http://docs.hpc.shef.ac.uk/en/latest/sharc/software/libs/cuda.html).

    nvcc helloworld.cu -gencode=arch=compute_37,code=compute_37 -gencode=arch=compute_60,code=compute_60 -o helloworld

Assuming there are no errors, your code will be built and you will have a new executable file in the working directory.

## Running a Simple Example as a Job

You cannot run our compiled GPU code on the worker node as it has no GPUs (or CUDA driver). Instead you must submit the execution of the GPU accelerated CUDA program via the ShARC job submission system (using `qsub`). The normal command for doing this on a public GPU node (i.e. free nodes shared with everyone in the university) would be (Note: **don’t run the following, it is for reference only**).

    qsub -l gpu=1 -b y ./helloworld

The `–l` command allow us to request specific resources in this case a GPU (`gpu=1`). The `–b y` command specifies that you are submitting a binary file rather than a batch script. To avoid queuing on the main ShARC GPU nodes you can use the following extended version of `qsub` for this lab class:

    qsub -P gputest -q gputest.q -l gpu=1 -b y ./helloworld

This will run the jobs on a private job queue (`-q gputest.q`) which requires that your CICS username has access to a special project group (`-P gputest`). Either a K80 or P100 GPU will be selected depending on what resources are free. All participants of the module have been granted access for the duration of the semester. After executing the above command, you will be notified that your job has been submitted but you will not see the output of the executable in the terminal.

Rather than using this long `qsub` command each time you want to run this example, you can instead place the job submission options in a bash script file and submit this. Examine the file `helloworld.sh` which contains scheduler instructions lines prefixed with `#$` matching the command line command above. The scheduler configuration can now be omitted from the `qsub` arguments and you can submit the job using the following command instead:

    qsub helloworld.sh

The labs in this training course will provide submission batch scripts for you.

## Monitoring the Progress and Output of Your Job

To monitor your job, you can use the `qstat` command. i.e.

    qstat –u <your_cics_username>

This will output a table with your jobs. E.g.

    job-ID  prior   name       user         state submit/start at     queue                          slots ja-task-ID
    -----------------------------------------------------------------------------------------------------------------
    205629 0.00534 bash       co1pr        r     01/18/2017 14:24:55 interactive.q@testnode01.icebe     1
    205650 0.00000 helloworld co1pr        qw    01/18/2017 14:26:45                                    1

The status of your GPU job will initially be `qw` to indicate queued and waiting. It will then change to `r` whilst running and will disappear once complete. In the above example you can see both the interactive session and the queued gpu job submission. When complete the job submission will create an execution log file with a file name matching the binary or bash script and with a file extension of `.e<jobid>` (i.e. `helloworld.e205650`). You can view the contents of this log file using the `cat` command. E.g.

    cat helloworld.e205650

If the execution produced no errors then this file will be empty. Additionally the job completion will create a matching output log file with an extension of `*.o<jobid>` (i.e. `helloworld.sh.o205650`). The contents of which will contain any output which your program prints. It should look like the following:

    Hello World from Thread 0
    Hello World from Thread 1
    Hello World from Thread 2
    Hello World from Thread 3
    Hello World from Thread 4
    Hello World from Thread 5
    Hello World from Thread 6
    Hello World from Thread 7
    Hello World from Thread 8
    Hello World from Thread 9

## Choosing GPU Architecture

If you wish to run your GPU code on a specific architecture (K80 or P100) then you can add the `gpu_arch` command to your `qsub` or batch file configuration. I.e.

    qsub -l gpu_arch=nv-k80 helloworld.sh

Or

    qsub -l gpu_arch=nv-p100* helloworld.sh

Note: that the gputest que has considerably more K80 nodes so you may find that your jobs run sooner if you target the K80 architecture. If you do not specify an architecture then the scheduler will pick an architecture for you but you may get different architectures for different job submissions.

## Profiling

In order to profile your code on ShARC you will need to run your code through `nvprof` to generate a timeline and analysis metric files for inspection within the visual profiler. You should generate the timeline and analysis date separately by running the following commands inside your bash script. Note: You will need to load the CUDA module on the GPU node to ensure that `nvprof` is accessible.

    module load libs/CUDA
    nvprof -o timeline.nvprof ./myapplication
    nvprof --analysis-metrics -o analysis.nvprof ./myapplication

E.g. a complete shell script for profiling might look like the following.

    #!/bin/bash
    # Request GPU resource
    #$ -l gpu=1
    # Use the gputest project group
    #$ -P gputest
    #Use the gputest queue
    #$ -q gputest.q

    module load libs/CUDA

    # Call the hello world executable
    nvprof -o timeline.nvprof ./helloworld
    nvprof -o analysis.nvprof --analysis-metrics ./helloworld

You can either inspect the profiles with the CUDA visual profiler on your local machine or from ShARC using your interactive session. To inspect the profiles on your local Windows machine you will need to copy the profiles back to your machine (e.g. using FTP) and run the profiler from the start menu. To view the profiles remotely from ShARC you will need to ensure that you have an X server running on your local (not ShARC) machine. If you are using a CICS managed desktop machine then run Xming. Assuming you have started an interactive session on a CPU worker node with `qrshx` and your ssh session with the –X argument (in the Putty configurations) you can use X forwarding to start the visual profiler by simply calling the `nvvp` program.

You should import your profile files by selecting File->Import. Select single process and then locate the separate analysis and timeline files.

## Building Solution Lab Examples

All of the COM4521 lab solutions include a Makefile which can be used for building the solutions in Linux and these have been tested on ShARC. The makefile will perform compilation and contains all of the `nvcc` arguments. You should try downloading various lab classes and trying out the profiler on them as well as on your own assignment. Feel free to also profile on the windows machines without using ShARC.
