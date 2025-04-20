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
![Alt text](<xv6-public/Screenshots/project3/make clean.png>)

6. After that execute  `make qemu-nox` which take us to xv6.
![Alt text](<xv6-public/Screenshots/project3/make qemu-nox.png>)

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
$ cat j.txt
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

# Project 3: Scheduler Implementation
## Variables and System Calls
To Implement both FCFS and Priority scheduler, we are using some common variables and system call.
1. Variables :  Declared Variable to keep track of things like number of process , Turn around time, waiting time and scheduler_pol1. `scheduler_pol1` is used to know the type of scheduler.

Added two variables in PCB(proc.h) there are priority and start_time which store the time when process got the CPU
2. Arrays : Declared arrays called stats, process and details. This arrays helps to print the executed process details in tabular format.

3. System Calls:

    a) set_scheduler_pol() : This is the system call used to set the variable `scheduler_pol1`, if the input is FCFS, we are making this variable as 1 else if the input is PRIO then it will be 2. This variable is used in scheduler() system call while selecting the process to assign CPU

    b) reset_scheduler_pol() : This is the system call use to print the details which are tracked in variables and arrays. After printing the details, we have the reset the variables to zero. This ensures that variables declared are brand new for next command.

    c) set_priority() : For priority based scheduler, we use this system call to set the priority for each process, once after created.
## Part - 1 First Come First Serve Scheduler

This is the command format use to execute FCFS

    Format  : scheduler FCFS <command1> # <command2> # <command3> ....
    Example : scheduler FCFS headuser j.txt # uniq j.txt # head j.txt

### User Level :</br>
This user level logic is implemented in scheduler.c file.
Based the input command, we start executing by checking argv[1] is "FCFS" or not. If it FCFS, we call set_scheduler_pol by 1 as an argument.

The commands are seperated by delimiter '#'. For that, we are iterating through argv in search of delimiter element by element. if the element is not '#' then we are storing that value in a temporary array(temp_cmd), if the element is '#' then we are creating the child process by call fork(). For created child we are executing `exec` command which is stored in temporary array. In this way, till '#' occur storing the command and when it occurs we are creating a children to execute that command. Here, for 'n' commands we are creating 'n' children.

Below is the code showing how we are splitting the main command based on delimiter and executing
```
while(i<=argc)
{
    if(i==argc || strcmp("#",argv[i])==0)
    {
        child++;
        if (fork() == 0)
        {
            // Child process: execute the command
            exec(temp_cmd[0], temp_cmd);
            // If exec fails, print an error message
            printf(1, "Error: Unable to execute %s\n", temp_cmd[1]);
            exit();
        }

        j=0;
        for (int k = 0; k < 10; k++)
        {
            temp_cmd[k] = 0;
        }
    }
    else
    {

        temp_cmd[j]=argv[i];
        j++;

    }
    i++;

}
```


Once after creating the children, parent(here scheduler) has to wait until every children report back and  child processes are waiting for scheduler() system call to assign the CPU.

After children reports back to parent, it executed reset_scheduler_pol and exits the user program.

### Kernel Level

In kernel level, for FCFS we modified only scheduler() system call. Based on  `scheduler_pol1` values we select scheduler for the processes waiting in queue for CPU.

we are iterating throught ptable where child processes created in user level are waiting. Based on the <b>arrival_time</b> we assign cpu to process, for that every time we have to iterate ptable and find the process with minimum arrival time and assign it to cpu. Completed process exits and reports to parent. Until every process finish exit this iteration continues.

Below are the xv6-public/Screenshots:<br>
1. Kernel commands followed by user commands <br>
`scheduler FCFS head j.txt # uniq j.txt # uniquser j.txt # headuser j.txt`

    ![Alt text](xv6-public/Screenshots/project3/FCFS/FCFS_kernel_user.png)

2. User commands followed by kernel commands <br>
`scheduler FCFS headuser j.txt # uniquser j.txt # head j.txt # uniq j.txt`
    ![Alt text](xv6-public/Screenshots/project3/FCFS/FCFS_user_kernel.png)
## Part - 2 Priority Scheduler

For priority scheduler, we have two types
1. With priorities mentioned explictly <br>
```
Command Format : scheduler PRIO -o <command1> priority1 # <command2> priority2 # ....
Example : scheduler PRIO -o head j.txt 4 # uniq j.txt 1
```
Here -o flag shows that priorities are mentioned explictly in command

2. Without priorities in command <br>
```
Command Format : scheduler PRIO <command1> # <command2> # ....
Example : scheduler PRIO head j.txt # uniq j.txt
```

### User Level :
Same as FCFS, for priority also we start with calling `set_scheduler_pol` call

I used a flag(without_priority) variable to differentiate between two types. We are setting flag variable by checking argv[2] is "-o" or not.

For type 1,
- split the commands and stores in an array
- creating a child by executing fork()  and set the priority to PCB by calling `set_priority` which is mentioned in the command
- calling exec() by passing array


For type 2,
- split the commands and stores in an array
- creating a child by executing fork()  and set the priority to PCB by calling `set_priority`, here order mentioned by the user the priority order of execution and order to get CPU for the child process.
- calling exec() by passing array

At the end, we call reset_ scheduler_pol to reset all variables and print the details tracked

### Kernel Level :

In kernel level, for priority scheduler  we implemented a system call to set the priority to process PCB and modified the scheduler() system call

System call : set_priority
In this call we are retriving the PCB and setting the priority as shown below

```
int sys_set_priority(int)
{
    int pri;
    struct proc *curproc = myproc();
    if(argint(0, &pri)<0)
    {
      return -1;
    }
    curproc->priority = pri;
    return 0;
}
```
In scheduler() call, when the `scheduler_pol1` is 2 it executed priority scheduler. Here also, we iterate through ptable but  we have search for a process with less priority and assign CPU to it.



Below are the output xv6-public/Screenshots and commands
1. scheduler PRIO -o head j.txt 2 # uniquser j.txt 3 # uniq j.txt 1 # headuser j.txt 4
    ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_num_kernel_high.png)
2. scheduler PRIO -o head j.txt 4 # uniquser j.txt 1 # uniq j.txt 3 # headuser j.txt 2
    ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_num_user_high.png)
3. scheduler PRIO -o head j.txt 4 # uniq j.txt 1 # headuser j.txt 3 # uniquser j.txt 2
    ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_num_kernel_user.png)
4. scheduler PRIO -o head -n 3 j.txt 4 # uniq -d j.txt 7 # headuser -n 2 j.txt 6 # uniquser j.txt 5 #
    ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_num_with_options.png)
5. scheduler PRIO -o head j.txt 4 # uniq j.txt 1 # ps 3 # uniquser j.txt 2
    ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_num_with_ps_variation.png)
6. scheduler PRIO head j.txt # uniq j.txt # headuser j.txt # uniquser j.txt
   ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_wo_num_kernel_user.png)

7. scheduler PRIO headuser j.txt # uniquser j.txt # head j.txt # uniq j.txt
    ![Alt text](xv6-public/Screenshots/project3/PRIO/PRIO_wo_num_user_kernel.png)


## Calculating and Tracking
while processes are exiting the system, we are calcuting all the measures and storing them in stats

arrival_time is assigned  while creating the process in fork().
start_time is assigned in scheduler() before CPU assigned to it.
end_time is assigned  in exit() call.


```
stats[0][num_of_process]=curproc->pid;
stats[1][num_of_process]=curproc->arrival_time;
stats[2][num_of_process]=curproc->start_time;
stats[3][num_of_process]=curproc->end_time;
stats[4][num_of_process]=curproc->end_time - curproc->start_time;
stats[5][num_of_process]=curproc->start_time-curproc->arrival_time;
stats[6][num_of_process]=curproc->end_time - curproc->arrival_time;
```

To keep processes name in tracking, we are using process array
```
char *temp_name=curproc->name;
  int i;
  for (i = 0; temp_name[i] != '\0'; i++)
  {
        process[num_of_process][i] = temp_name[i];
  }

```


## resources

1. https://www.geeksforgeeks.org/xv6-operating-system-add-a-user-program/
2. https://www.geeksforgeeks.orgxv6-operating-system-adding-a-new-system-call/
<br/>(This links helped me to implement the basic level of kernel and user program)

3. https://www.cse.iitb.ac.in/~mythili/os/
4. https://www.youtube.com/playlist?list=PLDW872573QAb4bj0URobvQTD41IV6gRkx
<br/>(I gone through this playlist from lecture 21 which gave me an understanding about trap, context switching, memory allocation in xv6 os level)

5. https://pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf
<br/>(I used this pdf to know the syntax and functionality of particular function or code)

6. https://www.gatevidyalay.com/first-come-first-serve-cpu-scheduling/
7. https://www.gatevidyalay.com/priority-scheduling-cpu-scheduling-examples/
8. https://www.gatevidyalay.com/turn-around-time-response-time-waiting-time/
9. Professor: Met professor during office hours, regarding implementation of priority based scheduler.

## Commands used in this project
```
scheduler FCFS head j.txt # uniq j.txt # uniquser j.txt # headuser j.txt

scheduler FCFS headuser j.txt # uniquser j.txt # head j.txt # uniq j.txt

scheduler PRIO head j.txt # uniq j.txt # headuser j.txt # uniquser j.txt

scheduler PRIO headuser j.txt # uniquser j.txt # head j.txt # uniq j.txt

scheduler PRIO -o head j.txt 4 # uniq j.txt 1 # headuser j.txt 3 # uniquser j.txt 2

scheduler PRIO -o head j.txt 4 # uniquser j.txt 1 # uniq j.txt 3 # headuser j.txt 2

scheduler PRIO -o head j.txt 2 # uniquser j.txt 3 # uniq j.txt 1 # headuser j.txt 4

scheduler PRIO -o head j.txt 4 # uniq j.txt 1 # ps 3 # uniquser j.txt 2

scheduler PRIO -o head -n 3 j.txt 4 # uniq -d j.txt 7 # headuser -n 2 j.txt 6 # uniquser j.txt 5

```


