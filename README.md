Using Great Lakes Cluster
=========================
# Introduction

If you find that you simulation or training code runs extremely slowly, using clusters is one option. However, the cost of HPC services on the market can be very high. You can easily spend thousands of dollars especially if running multiple large pieces of code. 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/28dc1843-c4ee-4f7f-9efd-d9975488d839)

Fortunately, the University of Michigan has several clusters available for our research. Among them, Great Lakes is the largest. 

> “The Great Lakes Slurm cluster is a campus-wide computing cluster that serves the broad needs of researchers across the university. The Great Lakes HPC Cluster replaced Flux, the shared research computing cluster that served over 300 research projects and 2,500 active users.”
Professor Okwudire has applied for a root account for our lab. In this case, we can use this powerful tool to boost our research. Before starting to introduce how to use it, understanding the architecture first is beneficial. This knowledge helps in fully utilizing the Great Lakes’ performance.

# Architecture of Great Lakes (or Other Clusters)

You might wonder what a cluster with 2500 active users looks like and how it manages to run so many tasks simultaneously. Unlike personal PCs,

> 'A computer cluster is a set of computers that work together so that they can be viewed as a single system.' 

We call a single 'computer' in the cluster a 'node'. A cluster node can be considered an individual computer with a CPU, GPUs, memory, and storage device on it. Note that a node has multiple cores that share memory and enable parallel computing. Essentially, the code can be run on cores on different nodes. Great Lakes uses [Slurm Workload Manager](https://slurm.schedmd.com/documentation.html) to organize the work that was submitted to the cluster. The figure below shows the architecture:

![Architecture](https://github.com/QuokeCola/gl_shell/assets/43491767/e7ecb74c-bb01-429c-9c87-7856b201c8ad)

The biggest advantage of this system, compared to our PC, is parallel computing. For a cluster, it is common to use server processors (like Intel XEON or AMD EPYC) with more cores than the same generation personal computers. Thus, when running a multi-process program, this enables a single node to outperform personal computers. Furthermore, the high-speed local network in the cluster system allows nodes to communicate with each other and perform larger-scale computations.

**Performing parallel computations on a single node with multiple cores is easier than using multiple nodes.** Please keep this in mind because this document will introduce how to use Great Lakes cluster step by step.

# First Glance & Simple Example

First, we need to create a cluster login account through this [[link](http://arc-ts.umich.edu/login-request)]. Simply enter you information, and the IT team will create an account for you in several days.

Once you have you cluster login account, you can check you account status on these pages: 

- [ARC Portal](https://portal.arc.umich.edu/) (To check the remaining resources, group members, the SLURM account you get access to)
- [Great Lakes Dashboard](https://greatlakes.arc-ts.umich.edu/pun/sys/dashboard/) (Your Great Lakes cluster interface, for submitting/checking you tasks, checking files).

The second link will be our most used link for submitting and managing our jobs. When you login, you will find a webpage that allows you to manage you job on great lakes cluster.

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/d9cddba4-7544-4aa9-9ef6-addd63196138)

In this tutorial, we will create a job combined with GUI and command line.
## Prepare Files
First, click on “File” -> “Home Directory”. This webpage shows you files in Great Lakes cluster. You can upload the code you want to run on Great Lakes here. However, simply uploading the code will not make Great Lakes run the code. In this case, we will compose a bash script for SLURM to make Great Lakes execute our code. 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/3059a270-0e4a-4aee-b52d-04b2a5d71627)

First, click on "New Directory" and name it as "test". Click on the newly created folder to enter it.

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/50b821da-7c14-425f-8d40-f69c7b31b655)

Click on "New File" button and type `test.sh`. A `test.sh` text file will be created. Click on `***...***` button on the right of the file and then click "edit". A online editor will open up and you can edit you test.sh here. Copy and paste these code into the file: 
## 
```sh
#!/bin/bash
#SBATCH --account=okwudire0 
#SBATCH --time=00:30:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=1000m
#SBATCH --job-name=YOUR_JOB_NAME
#SBATCH --mail-type=ALL

########### Copy you file to node
cp ./hello_matlab.m $TMPDIR
########### Move to node directory
cd $TMPDIR

########### Load matlab, run the code
module load matlab
matlab -nodisplay -nosplash -nodesktop -r "run('hello_matlab.m');exit;"

########### Copy the file on the node.
cp results.mat $SLURM_SUBMIT_DIR
```

Similarly, we created a new matlab file called `hello_matlab.m` and paste the scripts below into it:

```matlab
clear;clc;close all;
fprintf("Hello, world!");
text = sprintf("Hello, world!");
save("results.mat","text");
```

The matlab code print "Hello, world!" in command line and saved a "Hello, world!" string in `results.mat` file.

The result directory should looks like this: 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/5e9b0586-50db-4619-9eb0-a6f20c34ce4f)

## Submitting the Job
Then we click on the "Open in Terminal" button.
![image](https://github.com/QuokeCola/gl_shell/assets/43491767/1166d0a4-14be-4ca1-bf09-a261f3e1444c)

Use you UM password to login. Then you also need to pass DUO two-factor identification. After that, type this in command line and hit enter:

```sh
dos2unix test.sh && sbatch test.sh
```

You will find `Submitted batch job xxxxx`. This means our job has submitted to the Great Lakes! After a few minutes, you can find the `result.mat` file here, as well as a `slurm-xxxxx.out` file. This means that you job has been successfully executed. 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/75af4323-f232-4efe-b665-1ecb0207f5b3)

In "Jobs" -> "Active Jobs", you can also find the submitted job:

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/586b89f6-4c73-44f4-8f7b-322fc5695b5e)

# How it works?

The MATLAB code is quite simple. It just printed "Hello, world!" in command line and generate a `result.mat` file. The `test.sh` and the commandline are the keys of submitting our job appropriately. We will analyze them one by one. 

What does `test.sh` do? We can divide the test.sh into two portion:  
1. `#SBATCH` commands
2. Execution part

Note that the first line,`#!/bin/bash` just tell the cluster what types of shell we will use. For some scripts, `#!/bin/sh` will be used. This can be regard as a standard format. We can leave this unchanged. 

## SLURM `#SBATCH` Commands

```sh
#!/bin/bash
#SBATCH --account=okwudire0 
#SBATCH --time=00:30:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=1000m
#SBATCH --job-name=YOUR_JOB_NAME
#SBATCH --mail-type=ALL
```

Remember that the Great Lakes uses SLURM as the workload manager. SLURM can be regard as a factory manager. When submitting the code, you need to tell the manager how many workshops (node), how many workers (cores) and how long you will need to execute the code. 

These lines start with `#SBATCH` are for telling the manager (SLURM) the information "how many workshops (node), how many workers (cores) and how long you will need to execute the code". `salloc` command in CLI accept the same parameters. For the full list of `salloc` parameters, please check this [[link](https://slurm.schedmd.com/salloc.html)]

Please note that the more resources you job take, the harder for the manager to arrange you job into appropriate slot time and usually you job will have longer wait time.

Now we will analyze the code line by line.

### `#SBATCH --account=okwudire0`
`--account=xxx` tells SLURM which account is used to submit the code so it can charges from that account. `okwudire0` is our group's SLURM account. Another option is `engin1`. `engin1` is a shared account for Engineering department. You can check the [[link](https://caen.engin.umich.edu/rc/account/coe-shared/)] here for more details. Basically `engin1` has more restriction.

### `#SBATCH --time=00:30:00`

`--time` tells SLURM how long the code will execute. Note that when the wall time was hit, SLURM will kill the process and maybe you could not get the result (if the code is only going to produce the result at the end). However, really long wall time might make you code queued for a long time becasue SLURM could not find a good time slot for you job. So make good estimation of the job how long it will run. 
> Acceptable time formats include `minutes`, `minutes:seconds`, `hours:minutes:seconds`, `days-hours`, `days-hours:minutes` and `days-hours:minutes:seconds`.

### `#SBATCH --nodes=1`
`--nodes` tells SLURM how many nodes (computers) you will use. For now, we will use 1. To run a job with cross nodes, you need to use some system level packages suchs as MPI, which will be mentioned later.

### `#SBATCH --ntasks-per-node=1`

`--ntasks-per-node` specifies the number of cores SLURM needs for a node. The cores, sharing the same memory, can execute code in parallel. Core level parallel computing can be implemented by code. For example, in C++, `std::thread thread_object (callable);` can be used to create multi-threaded programs. Threads can run on different cores to expedite code execution.

### `#SBATCH --mem=1000m`

`--mem` indicates the amount of memory required by SLURM. Insufficient memory may cause you code to terminate and the job to fail.

### `#SBATCH --job-name=YOUR_JOB_NAME`

`--job-name` tells SLURM the name of you submitted job. The job you submitted will show in "jobs->active jobs" page. Choose any job name you prefer, but ensure it is clear and not confusing.
![image](https://github.com/QuokeCola/gl_shell/assets/43491767/2d157098-f1f1-4ae7-bac3-0c8f76641873)

### `#SBATCH --mail-type=ALL`
The Great Lakes system will send you emails regarding you job's status. `--mail-type` tells SLURM the types of events that should trigger notifications for this job. 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/e2bca367-ec12-4ee3-b225-9f76943095da)

<blockquote>

`--mail-type=<type>`

Notify user by email when certain event types occur. Valid type values are `NONE`, `BEGIN`, `END`, `FAIL`, `REQUEUE`, `ALL` (equivalent to `BEGIN, END, FAIL, INVALID_DEPEND, REQUEUE, STAGE_OUT`), `INVALID_DEPEND` (dependency never satisfied), `STAGE_OUT` (burst buffer stage out and teardown completed), `TIME_LIMIT`, `TIME_LIMIT_90` (reached 90 percent of time limit), `TIME_LIMIT_80` (reached 80 percent of time limit), and `TIME_LIMIT_50` (reached 50 percent of time limit). Multiple type values can be specified in a comma-separated list. Specify the user to be notified using `--mail-user`.

`--mail-user=<user>`

User to receive email notification of state changes as defined by `--mail-type`. The default value is the submitting user.
</blockquote>

[[source](https://slurm.schedmd.com/salloc.html)] -- In the middle of page

## Execution Part
```sh
########### Copy you file to node
cp ./hello_matlab.m $TMPDIR ### --- This is line 11.
########### Move to node directory
cd $TMPDIR                  ### --- This is line 13.

########### Load matlab, run the code
module load matlab          ### --- This is line 16.
matlab -nodisplay -nosplash -nodesktop -r "run('hello_matlab.m');exit;" ### --- This is line 17.

########### Copy the file on the node.
cp results.mat $SLURM_SUBMIT_DIR ### --- This is line 20.
```
The execution part is mainly a series of bash shell commands that will run on the nodes. It shares the same syntax in "Terminal" or "Command line", but was written in a file and the node will execute them serially.

Note that the when the job starts running, the node will first arrives at the submission folder (where you "Open in Terminal"). Then it will follows the commands one by one. In line 11, `cp` command will copy the file from you submission folder to a temporary folder on the node, for faster access. (When node get access to you file, it is through a local network and it is slower than on its own drive)

In line 13, `cd $TMPDIR` command will let the node enter the temporary directory. In line 16, `module load matlab` load the matlab package to the node shell instance. Then in line 17, the node call matlab to execute the "hello_matlab.m" script, where generate a `result.mat` file. Finally, in line 20, `cp results.mat $SLURM_SUBMIT_DIR` copy the `result.mat` file in `$TMPDIR` back to the submission folder. 

# Review - the Process

The following comic visualized the submission and execution process:

![SLURM-1](https://github.com/QuokeCola/gl_shell/assets/43491767/97a3851f-82fe-47db-8fa6-2b3cc5bb827a)

![SLURM-2](https://github.com/QuokeCola/gl_shell/assets/43491767/6fc73c7c-d63b-4e37-bb8f-211fee88c391)

![SLURM-3](https://github.com/QuokeCola/gl_shell/assets/43491767/acf72320-cc6c-4559-b83e-7ced8676110e)

![SLURM-4](https://github.com/QuokeCola/gl_shell/assets/43491767/51fd205e-b9dd-4ceb-844d-48e788e7f3da)

![SLURM-5](https://github.com/QuokeCola/gl_shell/assets/43491767/910d5d0e-ec47-4406-8add-5fe5d7f4af29)


## slurm-xxxxx.out file

You may noticed that, when you code start executing, here is an extra `slurm-xxxxxxx.out` file. This file will show the command line output of the job, allows you to track the process in realtime. Remember that there is a `fprintf("Hello, world!");` command in our submitted MATLAB code. If you check the `slurm-xxxxx.out` file, you will find it is recorded.

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/c41e0f15-d29a-44a1-9dc9-24bd7ec62590) 

# How to Make Good Use of this Tool (Tuning technique)

[TODO: TO BE IMPLEMENTED]

# Cross Node Processing - MPI libraries

> MPI is an interface which enables us to create multiple processes to be run on a single machine or on a cluster of machines, and enables message passing or in short sorts of communication between processes. [[source](https://scicomp.stackexchange.com/questions/34915/why-would-you-need-frameworks-like-mpi-when-you-can-multi-task-using-threads#:~:text=MPI%20is%20an%20interface%20which,sorts%20of%20communication%20between%20processes.)]

If you have the need of parallel computing cross nodes, MPI is the ideal tool. Here are different types of MPI library. For the Great Lakes cluster, it is using opemmpi 4.1.6. 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/c058dacf-26ae-4f85-acce-6b7a8027d2ed)

* Before start, you need to make sure you program is able to use MPI library. If it is not supported, even if we have multi-node hardware have MPI enabled, you program will not magically let two nodes corporate together.

To enable the MPI environment, you need to configure the following SLURM configuration (In you `.sh` SLURM configuration file).

1. `#SBATCH --nodes=2` For example, if you need 2 nodes, revise the `--nodes` parameters to `2` with this command. 

2. `#SBATCH --ntasks-per-node=8` This configures the number of cores of each node. Make sure you program enables multi-processing in node level.

3. `#SBATCH --mem=1000m` This allocate the memory of each node.

* Note that in this case you final price will be the <number of cores you applied> times you running time. For example, if you have applied total 16 cores and you ran you program for 1 hours, even if you are utilizing 1 node on 1 core, you price will be `1(hour)*16(cores)=16(hours)`, and 16 hours will be deducted from `okwudire0` account. So please make sure you can make good use of all the cores.

4. Then, we need to include the MPI library by adding this line after `#SBATCH` commands. `module load clang/2022.1.2 gcc/10.3.0 intel/2022.1.2 && module load openmpi/4.1.6`. This line loads the openmpi dependencies and then loads the openmpi itself.

5. Revise the execution library. Note that previously, we copy our code to `$TMPDIR` and execute the code there. However, in MPI this will lead to significant performance drop due to IO. This is because it takes extra time for two nodes to get access to `$TMPDIR`. Now we need to replace `$TMPDIR` with `$MPI_HOME`. This directory is optimized for MPI allows both nodes has fast access to the file.

## Find MPI and its dependencies

In the future, there might have some updates of Great Lakes system and the version of openmpi and the dependent libraries will change. To check the available openmpi, type `module spider openmpi` and it will list all the installed openmpi versions. 

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/74506ab0-fdb1-467b-8d01-0a0f03afe09a)

To check the dependency of the module, type `module spider openmpi/x.x.x`, and replace `x.x.x` to the version you want to add, where was listed with the `module spider openmpi` command. For example,

```sh
module spider openmpi/4.1.6
```

When you use `module spider` with a specific openmpi version, you will find its dependencies at here:

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/e08c85a8-f24a-4993-8e0c-c0b75f0b1395)

Just use the `module load <package1> <package2> <package3> <package4 ...>` command and replace the `<packages>` with the module you found. For example, 

```sh
module load clang/2022.1.2 gcc/10.3.0 intel/2022.1.2
```

## Making Good Use of All Cores

When a job is submitted to SLURM, it is usually hard to get the performance information and determine if it is using all cores efficiently in realtime. Thus, it is strongly recommended to use `interactive session` to tune you program.

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/e743a0d8-72f2-4275-8063-63b28eb84330)

In this case, you can launch one terminal to run you code, and launch another terminal to monitor the performance by typing `top` command.

![image](https://github.com/QuokeCola/gl_shell/assets/43491767/8afe629e-f621-4c36-8856-b383c92dfcc8) 

At here you can check `%CPU` section to decide how the CPU resource was utilized. the 100% means you program is fully utilizing 1 single core. If you are multithreading, the %CPU might exceed 100 as you are using more than 1 core. However, if you are using MPI, you process might be divided into several items in `top` so you need to sum up the CPU utilization and check if it conforms with the number of cores.
