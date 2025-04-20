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
![vs code](xv6-public/Screenshots/project1/screenshot_open_vs.png)

5. Execute `make clean`.
![Alt text](xv6-public/xv6-public/Screenshots/project1/vscode_make_clean.png)

6. After that execute  `make qemu-nox` which take us to xv6.
![Alt text](xv6-public/xv6-public/Screenshots/project1/vscode_make_qemu-nox.png)
7. We can proceed with the commands given below.
#### In Ubuntu
1. Open Ubuntu
2. change the path to xv6-public by executing `cd` command.
3. Execute `make clean`.
![Alt text](xv6-public/Screenshots/project1/make_clean.png)

4. After that execute  `make qemu-nox` which take us to xv6.
![Alt text](xv6-public/Screenshots/project1/After_exe_qemu-nox.png)
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


# Implementing uniq on xv6

## Part 1 Task-1 - User space

To Execute uniq command in User space, we need to modify two files:
 1. Makefile
 2. uniquser.c

In Makefile, I added `_uniquser\` in UPROGS variable and `uniquser.c` in EXTRA variable inorder to execute userlevel program(uniquser.c).
Even after executing  `make clean` txt files should not be deleted for that we have to add `*.txt` to fs.img variable.

uniquser.c is the file where main logic resides

### Command used are :
    1.cat OS611_example.txt | uniquser
    2.uniquser OS611_example.txt
    3.uniquser -c OS611_example.txt
    4.uniquser -d OS611_example.txt
    5.uniquser -i OS611_example.txt

Based on number of arguments(argc) in command they are 3 variation which are implemented in switch case.
1. Number of Arguments = 1 :

    If argc == 1, setting the bool_stdin = 1, which means input should be read from standard input. For this command `cat OS611_example.txt | uniquser`, standard input is the content present in OS611_example.txt. While declaring the filedescriptor(filedesc), i checking the bool_stdin value. if it's '1' then i'm returning '0' because for standard input, filedescriptor value should be 0. Later it executes the main logic which is explained below

    After executing the command `cat OS611_example.txt | uniquser` output is as shown below.

    ![user level uniq with single arg](xv6-public/Screenshots/project1/uniq/uniquser_arg_cat.png)



2. Number of Arguments = 2:

    If No.of arguments is 2, this means first argument is 'uniquser' itself and second argument is input file, on which uniq operation should done. During filedescriptor declaration, this filename is passed to 'open' system call along with 'O_RDONLY' argument. With this filedescriptor, 'read' system call reads the contents character by character in main logic.

    Output for the command `uniquser OS611_example.txt` is

    ![user level uniq with filename](xv6-public/Screenshots/project1/uniq/uniquser_filename.png)

3. Number of Arguments = 3:

    Here second argument is option(-c/-d/-i), first and third are 'headuser' and input file respectively.In this case, we set the 'options' variable to 1,2,3 for options c,i,d based on `argv[1][1]` i.e., second argument second character as the second argument first character is '-'. This options variable is used while printing the output.

    The outputs of commands with three arguments is as below

    ![user level uniq for all options](xv6-public/Screenshots/project1/uniq/uniquser_allOptions_snap.png)

    for option  -c, output prints as count of same line (repeats_cnt variable) and line.
    for option  -d, this print only duplicate lines ( repeats_cnt>1 )
    for option  -i, ignores the case while comparing the lines, in above screenshot we can observe that even thought first two lines are different in case. this option ignored the case while comparing


Main Logic : After setting required variables based on number of arguments in switch case. it starts reading the file character by character, after reading ever single character that character is copied to current_line variable till '\n' occur. After encounting '\n', it compare current_line with previous_line, if previous_line is same current_line goes to next line after incrementing repeats_cnt variable. if they are not same, we are printing previous_line based on the given option in command.

#### Error Handling
1. if the options are out of scope i.e., other then c,i,d. output prompts as 'Error : Invalid option'
2. if second argument is not starting with '-'. output prompts as 'Error: Invalid argument'
3. if command is not given correctly, 'Usage: uniquser [-c]/[-i]/[-d] filename' is prompted

## Part 1 Task-2 - Kernel space
To Execute uniq command in kernel space, we need to modify five files:
1. syscall.h
2. syscall.c
3. usys.s
4. user.h
5. sysfile.c


1.syscall.h : Adding uniq `#define SYS_uniq 23` to already defined syscalls, to implement the syscall in kernel level.

2.syscall.c : we have to add an element to syscall array `[SYS_uniq]  sys_uniq` and we have declare the function as extern

3.usys.s : As we are implementing system call `SYSCALL(uniq)` have to be added in usys.s

4.user.h :  Here we have to declare the function definition clearly with number of arguments and return type.

5.sysfile.c : System call logic is similar to user level which is written in `int sys_uniq()` function. System call takes filedescriptor, options and argc values as arguments, we will create a struture pointer for current process and we create the file pointer from ofile array(open files array in PCB) with the help of filedescriptor value. In kernel level, inorder to read file it takes ip(instruction pointer),buffer,offset,size of buff. We will get ip with file pointer created earlier and before using this ip pointer we have to lock it, preventing other processes to access that and at end of system call we have to unlock the ip pointer.

### Command used are :
    1. cat OS611_example.txt | uniq
    2. uniq OS611_example.txt
    3. uniq -c OS611_example.txt
    4. uniq -d OS611_example.txt
    5. uniq -i OS611_example.txt

<br/>

After executing `cat OS611_example.txt | uniq` the output is as shown below.

![Kernel level uniq with single arg](xv6-public/Screenshots/project1/uniq/uniq_arg_cat.png)

<br/>

For `uniq OS611_example.txt` command this is the output

![kernel level uniq with filename](xv6-public/Screenshots/project1/uniq/uniq_filename.png)

<br/>
Outputs for the commands with 3 arguments are as shown below.

![user level uniq for all options](xv6-public/Screenshots/project1/uniq/uniq_allOptions_snap.png)

<br/>

#### Error Handling
1. After reading the arguments checking anyone of them are less than 0, if any argument is less than 0, we will return -1.
2. After creating a file pointer from current process with filedescriptor, checking whether file pointer created is correct or not.

# Implementing head on xv6

## Part 2 Task-1 - User space

To Execute head command in User space, we need to modify two files:
 1. Makefile
 2. uniquser.c

In Makefile, I added `_headuser\` in UPROGS variable and `headuser.c` in EXTRA variable inorder to execute userlevel program(headuser.c).

headuser.c is the file where we have all the code

### Command used are :
    1. headuser
    2. headuser OS611_example.txt
    3. headuser OS611_example.txt OS611_example_1.txt
    4. headuser -n 3 OS611_example.txt
    5. headuser -n 3 OS611_example.txt OS611_example_1.txt



Based on number of arguments(argc) in command they are 4  variation which are implemented in if else.
1. Number of Arguments = 1:

    If number of arguments is 1 then we have to take 14 lines of user input and print. As we have to read from standard input, we pass '0' to read system call along with a buffer and size of buffer. By default it take 14 inputlines from user as shown the below image.

    ![only head in user mode](xv6-public/Screenshots/project1/head/headuser_only.png)

3. Number of Arguments >= 2:

    if No.of arguments are greater than 2 and there are no options in command means first argument is 'headuser' and from second argument they are filenames. So, from second argument we will open all the files one by one and call the 'head_for_usermode' function with filedescriptor and no of lines to print i.e., here it's always 14 lines. Below are the xv6-public/Screenshots/project1, executing command for one file and multiple files.

    ![head in user mode with one file](xv6-public/Screenshots/project1/head/headuser_filename.png)
    <br/>

    ![head in user mode with multiple files](xv6-public/Screenshots/project1/head/headuser_multiple_files.png)

4. Number of Arguments >=4:

    if No.of arguments are greater than 4 means first argument is headuser itself, second argument is option(-n), third one is number of line and from four argument they are filenames. Here while passing arguments we pass filedescriptor and number of lines which is specified by user in command. Below are the xv6-public/Screenshots/project1, executing command for one file and multiple files.

    ![head in user mode with option and one file](xv6-public/Screenshots/project1/head/headuser-nOptions_filename.png)
    <br/>

    ![head in user mode with option and multiple files](xv6-public/Screenshots/project1/head/headuser-nOptions_multiplefiles.png)

head_for_usermode logic :
    This function takes filedescriptor and number of lines as arguments. In while loop we read the content from file and place in buffer(line variable). We iterate the buffer, character by character print them and along with we search for '\n' when we found that printedlines will be incremented. if printedlines exceeds number of linesthe condition will fail, loop ends there.

#### Error Handling
1. When ever we read the file, we will check number of bytes returned are greater than zero or not. if number of bytes returned are less than zero, it will error message.
2. If none of condition matched, output prompts as "Usage: head [-n N] inputfile"

## Part 1 Task-2 - Kernel space

To Execute head command in Kernel space, we need to modify five files:
1. syscall.h
2. syscall.c
3. usys.s
4. user.h
5. sysfile.c


1.syscall.h : Adding  `#define SYS_head 22` to already defined syscalls, to implement the syscall in kernel level.

2.syscall.c : we have to add an element to syscall array `[SYS_head]  sys_head` and we have declare the function as extern.

3.usys.s : As we are implementing system call `SYSCALL(head)` have to be added in usys.s

4.user.h :  we have to declare the function definition clearly with number of arguments and return type.

5.sysfile.c : logic is under `sys_head()` the logic is pretty much same as user level code but here we are store characters into an array(res) until condition fail whether file reaches the end or printedlines reached the limit which is mentioned by user. offset is incremented by size of bytes read from file for every iteration.

### Command used are :
    1. head
    2. head OS611_example.txt
    3. head OS611_example.txt OS611_example_1.txt
    4. head -n 3 OS611_example.txt
    5. head -n 3 OS611_example.txt OS611_example_1.txt


<br/>

After executing `head` the output is as shown below.

![Kernel level head only](xv6-public/Screenshots/project1/head/head_only.png)
<br/>

For `head OS611_example.txt` command this is the output

![kernel level head with filename](xv6-public/Screenshots/project1/head/head_filename.png)

For head with multiple files

![kernel level head with multiple filename](xv6-public/Screenshots/project1/head/head_multiple_files.png)
<br/>


Outputs for the commands with options are as shown below.
![kernel level head with options filename](xv6-public/Screenshots/project1/head/head-nOptions_filename.png)
<br/>

![kernel level head with options multiple filenames](xv6-public/Screenshots/project1/head/head-nOptions_filename.png)

<br/>

#### Error Handling
1. After reading the arguments checking anyone of them are less than 0, if any argument is less than 0, we will return -1.
2. After creating a file pointer from current process with filedescriptor, checking whether file pointer created is correct or not.


### Resources
1. https://www.geeksforgeeks.org/xv6-operating-system-add-a-user-program/
2. https://www.geeksforgeeks.orgxv6-operating-system-adding-a-new-system-call/
<br/>(This links helped me to implement the basic level of kernel and user program)

3. https://www.cse.iitb.ac.in/~mythili/os/
4. https://www.youtube.com/playlist?list=PLDW872573QAb4bj0URobvQTD41IV6gRkx
<br/>(I gone through this playlist from lecture 21 which gave me an understanding about trap, context switching, memory allocation in xv6 os level)

5. https://pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf
<br/>(I used this pdf to know the syntax and functionality of particular function or code)

6. Teaching Assitance :  we reached out to manas sanjay, when we are stucked with a doubt how to call both user and kernel code in single program.He suggested us to write two programs, one calling kernel and another one calling user function.
