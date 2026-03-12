# Recreating the System-Wide FD Tables

## Metadata
 * Author: Xiaoran Zhang
 * Date: March 11
 * Version: 1

## Introduction
This project implements a C program that displays the tables used by the operating system to keep track of open files, file descriptors (FD), and processes. By exploring the ```/proc``` virtual filesystem interface, the program retrieves real-time information about running processes and their open file descriptors.

The program can generate multiple tables:
* Process FD Table
* System-wide FD Table
* Vnodes FD Table
* A composed view of the previous table
* A table summarizing number of FDs open per user processes

Additionally, it supports threshold-based filtering and optional output saving in text or binary format.

## Description of how I solve/approach the problem
1. Use the ```/proc``` filesystem to access process directories and their fd subdirectories.
2. Store collected information in structured data types for easy access and manipulation.
3. Implement different display functions to present the same data in various formats based on user requirements (command line augment).
4. Check different flags to determine which tables to display and how to format the output.

## Implementation

This program is structured into several modules, each handling specific functionality:
### Modules:
#### Module 1: checkArgs
* purpose: Check and validate command-line arguments.
* Files: ```checkArgs.h```, ```checkArgs.c```
* Data Structure: ```Options```: Store all flag states and values.
* Functions:
  1. ```void check_arguments(int argc, char *argv[], Options *opt)```: Process CLA and updates the Options struct accordingly. It iterates through all arguments, recognizes flags, extracts threshold values, and identifies PID arguments. If no display flags are specified, it defaults to showing all tables.

#### Module 2: proc
* Retrieve process and file descriptor information from ```/proc```.
* Files: ```proc.h```, ```proc.c```
* Data Structure: ```FDInfo``` :Store PID, FD, inode, and filename.
* Functions:
  1. ```int get_pid_list(Options *opt, int pid_list[], int max_pids)```: If user specified a pid, returns 1. Otherwise, scans /proc directory for all numeric entries and stores them in pid_list.
  2. ```int get_process_fd_info(int pid, FDInfo table[], int idx)```: Collect information about all open file descriptors for a given process. 

#### Module 3: printTable
* purpose: Display tables on screen.
* Files: ```printTable.h```, ```printTable.c```
* Functions:
  1. ```void print_per_process(FDInfo table[], int size)```: Display PID and FD columns
  2. ```void print_systemWide(FDInfo table[], int size)```: Display PID, FD, and Filename columns
  3. ```void print_vnodes(FDInfo table[], int size)```:  Display FD and Inode columns
  4. ```void print_composite(FDInfo table[], int size)```: Display PID, FD, Filename, Inode columns.
  5. ```void print_summary(FDInfo table[], int size)```: Calculates and display the number of open file descriptors for each process, shown as "PID (count)".
  6. ```void print_threshold(FDInfo table[], int size, int threshold)```: Identify and list processes whose open FD count are larger than the specified threshold value.

#### Module 4: fileOutput
* purpose: Save composite table to files
* Files: fileOutput.h, fileOutput.c
* Functions:
    1. ```void get_txt(FDInfo table[], int size)```: Save composite table in ASCII format to compositeTable.txt.
    2. ```void get_binary(FDInfo table[], int size)```: Saves composite table in binary format to compositeTable.bin.
#### Module 5: main
```int main(int argc, char** argv)```
```main``` controls entire program flow: it initializes the Options structure, parses command-line arguments, allocates memory, retrieves process information, and calls the appropriate display and output functions based on the flags provided.

## Instructions in how to compile the code 
### use MakeFile!
* ```make```: Compile the whole project
* ```make clean```: Remove all object files and the executable

## Expected Results
### Valid Command line augment
* ```--per-process```, indicates that only the process FD table will be displayed (PID and FD)
* ```--systemWide```, indicates that only the system-wide FD table will be displayed (PID, FD, Filename)
* ```--Vnodes```, indicates that the Vnodes FD table will be displayed (FD, Inode)
* ```--composite```, indicates that only the composed table will be displayed (PID, FD, Filename, Inode)
* ```--summary```, indicates that a table summarizing number of FDs open per process will be displayed
* ```--threshold=X```, where X denotes an integer, indicating that processes which have a number of FD assigned larger than X should be flagged in the output.
* ```--output_TXT```, save composite table to compositeTable.txt
* ```--output_binary```, save composite table to compositeTable.bin
* ```[PID]```, process ID to examine

### Default Behavior
If no display flags are provided, the program defaults to showing all tables (--per-process, --systemWide, --Vnodes, --composite).
### Invalid CLA
The program silently ignores any unrecognized command-line arguments and continues processing valid ones.
### some examples
1. ```./showFDtables  --composite```
```
          PID     FD       Filename       Inode
         ========================================
0         319528  0       /dev/null       5
1         319528  1       socket:[55066739]       55066739
2         319528  2       socket:[55066739]       55066739
3         319528  3       socket:[55054408]       55054408
4         319528  4       anon_inode:[eventpoll]       1059
5         319528  5       anon_inode:[signalfd]       1059
6         319528  6       anon_inode:inotify       1059
7         319528  7       /sys/fs/cgroup/user.slice/user-27079605.slice/user@27079605.service       992447
8         319528  8       anon_inode:[timerfd]       1059
9         319528  9       /usr/lib/systemd/systemd-executor       8425457
10         319528  10       anon_inode:inotify       1059
...
575         346253  1       /dev/pts/2       5
576         346253  2       /dev/pts/2       5
577         346253  3       /proc/346253/fd       55390527
578         346253  22       /cmshome/zha16531/.vscode-server/data/logs/20260311T183013/remoteagent.log       5277345535
579         346253  23       /cmshome/zha16531/.vscode-server/data/logs/20260311T183013/ptyhost.log       5295336028
580         346253  25       /cmshome/zha16531/.vscode-server/data/logs/20260311T183013/remoteTelemetry.log       5277345536
```
2. ```./showFDtables 319712```
```

         PID     FD
        ============
        319712  0
        319712  1
        319712  2
        319712  3
        319712  4
        319712  5
        319712  6
        319712  7
...
        319712  23
        319712  24
        319712  25
        319712  26
        319712  27
        319712  28
        319712  30
        319712  31
        319712  33
        319712  39
        ============

         PID     FD      Filename
        ========================================
         319712  0       /dev/null
         319712  1       pipe:[55064891]
         319712  2       pipe:[55064892]
         319712  3       anon_inode:[eventpoll]
         319712  4       anon_inode:[io_uring]
         319712  5       pipe:[55064895]
         319712  6       pipe:[55064895]
         319712  7       pipe:[55064896]
...
         319712  28       socket:[55064906]
         319712  30       anon_inode:inotify
         319712  31       socket:[55064908]
         319712  33       socket:[55064910]
         319712  39       socket:[55061846]
        ========================================

           FD            Inode
        ========================================
           0              5
           1              55064891
           2              55064892
           3              1059
           4              55064894
           5              55064895
...
           25              5277345536
           26              55064904
           27              55064913
           28              55064906
           30              1059
           31              55064908
           33              55064910
           39              55061846
        ========================================

          PID     FD       Filename       Inode
         ========================================
0         319712  0       /dev/null       5
1         319712  1       pipe:[55064891]       55064891
2         319712  2       pipe:[55064892]       55064892
3         319712  3       anon_inode:[eventpoll]       1059
4         319712  4       anon_inode:[io_uring]       55064894
5         319712  5       pipe:[55064895]       55064895
...
29         319712  30       anon_inode:inotify       1059
30         319712  31       socket:[55064908]       55064908
31         319712  33       socket:[55064910]       55064910
32         319712  39       socket:[55061846]       55061846
         ========================================
```
3. Invalid [PID], e.g. ```./showFDtables 0000022```

```

         PID     FD
        ============
        ============

         PID     FD      Filename
        ========================================
        ========================================

           FD            Inode
        ========================================
        ========================================

          PID     FD       Filename       Inode
         ========================================
         ========================================
```
## Disclaimers!
THE PROGRAM WORKS ONLY WHEN:
* This program must be compiled and run on Linux systems with /proc filesystem available
* The maximum number of processes does not exceed MAX_PIDS (4096)
* The total number of file descriptors across all processes does not exceed MAX_FD (100,000)
* Filename paths do not exceed 511 characters
* The user has permission to read /proc/[pid]/fd directories for the processes being examined

## Bonus 
Run a comparison using these two flags, for a few cases, i.e. one PID and larger number (all PIDs for the user), time the run, you can use the time command from the shell and compare the file sizes.
For your comparison to be relevant should have some statistical support with a number of cases between 5 and 10, i.e. average the values in time and sizes over the number of times you run this and include average and standard deviations.
Include this information in your report and elaborate about the observations you notice and conclusions you can draw from them.
### case1
time ./showFDtables 349845 --output_TXT
ls -lh compositeTable.txt
```
real    0m0.014s
user    0m0.002s
sys     0m0.005s
-rw------- 1 zha16531 cscb09w26i 687 Mar 11 23:04 compositeTable.txt
```
time ./showFDtables 349845 --output_binary
ls -lh compositeTable.bin
```
real    0m0.025s
user    0m0.006s
sys     0m0.013s
-rw------- 1 zha16531 cscb09w26i 3.7K Mar 11 23:06 compositeTable.bin
```
### case2
time ./showFDtables --output_TXT
ls -lh compositeTable.txt
```
real    0m0.119s
user    0m0.007s
sys     0m0.034s
-rw------- 1 zha16531 cscb09w26i 48K Mar 11 23:09 compositeTable.txt
```
time ./showFDtables --output_binary
ls -lh compositeTable.bin
```
real    0m0.051s
user    0m0.005s
sys     0m0.033s
-rw------- 1 zha16531 cscb09w26i 326K Mar 11 23:09 compositeTable.bin
```
In the single PID case, the TXT output took 0.014 seconds and produced a file of 687 bytes, while the binary output took 0.025 seconds and produced a file of 3.7 KB. In the all-PID case, the TXT output took 0.119 seconds and generated a 48 KB file, whereas the binary output took 0.051 seconds and generated a 326 KB file. Overall, the binary output produced significantly larger files, while the execution times varied between the two modes depending on the case.

