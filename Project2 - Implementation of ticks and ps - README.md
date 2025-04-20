##### Note: For better view of text and images, please open in preview mode

### System Specification
Device name	: LAPTOP-MBO0EMU8 <br/>
Processor	: Intel(R) Core(TM) i5-10300H CPU @ 2.50GHz <br/>
Installed RAM : 8.00 GB (7.83 GB usable) <br/>
System type	: windows 64-bit operating system, x64-based processor <br/>

### How to run the code
#### In Visual studio code
1. Open visual studio code
2. Click on Terminal option, On top left corner and open a new terminal
3. In terminal, click on the arrow which is on side of '+' and select 'Ubuntu(wsl)'.
4. change the path to xv6-public by executing `cd` command.
5. Execute `make clean`.
![Alt text](xv6-public/Screenshots/project2/make_clean.png)

6. After that execute  `make qemu-nox` which take us to xv6.
![Alt text](xv6-public/Screenshots/project2/after_make_qemu-nox.png)
7. We can proceed with the commands given below(commands used this project are in the end to this readme file).

#### In Ubuntu
1. Open Ubuntu
2. change the path to xv6-public by executing `cd` command.
3. Execute `make clean`.
4. After that execute  `make qemu-nox` which take us to xv6.
5. We can proceed with the commands given below.

### Input files
Below are the files used for testing the code in both kernel and user level for uniq and head

```
$ cat OS611_example.txt
i UNDERSTAND OPERATING SYSTEM.
I understand Operating system.
I understand Operating system.
I understand Operating system.
I love to work on OS.
I love to work on OS.
Thanks xv6.
```

```
$ cat OS611_example_1.txt
This is OS project.
This is OS project.
Implementing head and uniq on XV6.
Implementing head and uniq on XV6.
```

# Implement various system call focused on process in xv6
## Part-1 Getting process statistics

For part 1, we have to modify two files:
 - proc.h
 - sysfile.c
 - proc.c
 - test.c
 - Makefile

1)<b>proc.h</b>: In 'proc' struct, i added 6 field. creation_time holds ticks when process starts, end_time holds ticks when process ends, total_time is the difference between end_time and creation_time, starttimestamp is the timestamp when process starts in UTC timezone, endtimestamp is the timestamp when process ends and last one is bool_test with this bool variable we are ensuring to print the stats only when we run 'test' command

2)<b>sysfile.c</b> : In this file, system call is implemented which makes bool_test variable to 1

3)<b>proc.c</b> : In fork system call, we are storing the ticks into creation_time when process is starting and also storing timestamp using `cmostime` into starttimestamp

In exit system call, we are storing the ticks into end_time when process is ending, difference between end_time and creation_time is stored in total_time and also storing timestamp using `cmostime` into endtimestamp. if bool_test is 1 then print all the stats.

In Stats, we are printing
-  process name
-  start time
-  end time
-  total time in different units (ticks, seconds, milli seconds)
-  time stamps of start and end

![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/head/head_filename.png)

Here is the example for the stats displayed and we can observe that start timestamp and end timestamp are same because the duration of process is very short(Just 30 milli seconds)

4)<b>test.c</b> : In this file we mainly do two things, first is to call `bool_conv` system call which changes the boolean value and second one is to call `exec` which replaces the memory of test to command which we want to know the stats.

5)<b>Makefile</b> : Added  `_test` in UPROGS and `test.c` in EXTRA

And modified all the files which we have to modify for adding system call.

I implemented in such a way that we can execute head and uniq seperately and together which also works for flags which are implemented in project1.outputs are as shown below :

1. cat OS611_example.txt | test uniq

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/uniq/uniq_cat_filename.png)

2. test uniq OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/uniq/uniq_filename.png)

3. test uniq -c OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/uniq/uniq-cOption.png)

4. test uniq -d OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/uniq/uniq-dOption.png)

5. test uniq -i OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/uniq/uniq-iOption.png)

6. test head OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/head/head_filename.png)

7. test head OS611_example.txt OS611_example_1.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/head/head_multiple_files.png)
8. test head -n 3 OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/head/head-nOptions_filename.png)
9. test head -n 3 OS611_example.txt OS611_example_1.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/head/head-nOptions_multiplefiles.png)

10. test head OS611_example.txt | test uniq OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/combination/uniq_head_1.png)

11. test uniq OS611_example.txt | test head -n 5 OS611_example.txt

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/combination/uniq_head_2.png)

12. test head -n 5 OS611_example.txt | ps

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/combination/head_ps.png)
13. test uniq OS611_example.txt | ps

    ![Alt text](xv6-public/Screenshots/project2/Part-1_Stats/combination/uniq_ps.png)


Above 10, 11 outputs for the command which executes both head, uniq and printed stats

### Error Handling
1. if arguments given are less than 2, it prompts 'Usage: test command'.
2. After executing `exec` system call it should no return to user program again, if in any case it returns to user program, it prompts 'exec failed'.

## Part-2 Implementing ps on xv6

For part 2, we have to modify three files:
 - proc.c
 - ps.c
 - Makefile

1.<b>proc.c</b> : In this file, we implemented sys_ps system call. Here we have 3 variation, first one is just `ps` we have to iterate over ptable and print the fields(PID, process name, Status, Start time, Total time).

![Alt text](xv6-public/Screenshots/project2/Part-2_ps/ps.png)

Secondly, `ps <PID>` which means while iterating throught the ptable, we have to print only the details of that PID. Below is the output of `ps 3` command.

![Alt text](xv6-public/Screenshots/project2/Part-2_ps/ps_pid.png)

Third one is filtering by pname while iterating we search by name of the process. `ps sh` command output is shown as below.

![Alt text](xv6-public/Screenshots/project2/Part-2_ps/ps_sh.png)

2.<b>ps.c</b> : We are calling ps system call based on arguments count, if we have only one argument we pass "NULL" else if we have two arguments then we pass 1st indexed argument along atoi(argv[1])

3.<b>Makefile</b> : Added  `_ps` in UPROGS and `ps.c` in EXTRA

And modified all the files which we have to modify for adding system call.

### Error Handling
1. If number of arguments are greater than 2 then we are prompts "Usage: ps PID/process_name/empty"
2. if PID or pname is not present in ptable then we print "PID/pname is exited or not present in ptable"


## resources

1. https://www.geeksforgeeks.org/xv6-operating-system-add-a-user-program/
2. https://www.geeksforgeeks.orgxv6-operating-system-adding-a-new-system-call/
<br/>(This links helped me to implement the basic level of kernel and user program)

3. https://www.cse.iitb.ac.in/~mythili/os/
4. https://www.youtube.com/playlist?list=PLDW872573QAb4bj0URobvQTD41IV6gRkx
<br/>(I gone through this playlist from lecture 21 which gave me an understanding about trap, context switching, memory allocation in xv6 os level)

5. https://pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf
<br/>(I used this pdf to know the syntax and functionality of particular function or code)

6. Teaching Assitance : we reached out to manas, in starting days of this project when we are not clear with task 1 objectives and expectations. After meeting him we got some idea of how to proceed.

7. Professor : Joined in teams call with Dr.templeton during his office hours and in this call we got to know how the command should(whether we have to include options and multiple files in command).




## Commands used in this project

### Part 1
#### Uniq in different variation
```
test cat OS611_example.txt | test uniq
test uniq OS611_example.txt
test uniq -c OS611_example.txt
test uniq -d OS611_example.txt
test uniq -i OS611_example.txt
```
#### Head in different variation
```
test head OS611_example.txt
test head OS611_example.txt OS611_example_1.txt
test head -n 3 OS611_example.txt
test head -n 3 OS611_example.txt OS611_example_1.txt
```
#### Combination of uniq and head
```
test head OS611_example.txt | test uniq OS611_example.txt
test uniq OS611_example.txt | test head -n 5 OS611_example.txt
```
#### Combination of uniq, head with ps
```
test head -n 5 OS611_example.txt | ps
test uniq OS611_example.txt | ps
```

### part 2
    ps
    ps 3
    ps sh
    ps init
