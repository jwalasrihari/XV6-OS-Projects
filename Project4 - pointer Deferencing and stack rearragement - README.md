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
![Alt text](<xv6-public/Screenshots/project4/make clean.png>)

6. After that execute  `make qemu-nox` which take us to xv6.
![Alt text](<xv6-public/Screenshots/project4/make qemu-nox.png>)

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



## Pointer Dereference

In xv6 before unmapping pages if we access null pointer then we will below trap. This type of trap occurs when the CPU encounters an instruction that it does not recognize or that is not a valid operation in the current context.
![Alt text](<xv6-public/Screenshots/project4/Pointer Dereference/accessing null pointer.png>)

In linux if we access null pointer this causes segmentation fault.
![Alt text](<xv6-public/Screenshots/project4/Pointer Dereference/linux null pointer accessing.png>)

To unmap three pages and start from 0X3000 we have modify below files<br/>
    1.exec.c <br/>
    2.vm.c   <br/>
    3.makefile.c

1. exec.c : Before loading user code/ program we have to change the sz to 3 * PGSIZE this means user code loading start from 0x3000

2. vm.c : In this code, while copy contents from parent to child we have to start from 3rd page. So, in copyuvm inside for loop i is assigned to 3*PGSIZE

3. makefile.c : In makefile we have to modify entry point as loading of program starts from 3rd page. Now the entry point is 0x3000

    ```
    %: %.o $(ULIB)
        $(LD) $(LDFLAGS) -N -e main -Ttext <mark> 0x3000 <mark/> -o $@ $^
        $(OBJDUMP) -S $@ > $*.asm
        $(OBJDUMP) -t $@ | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $*.sym

    _forktest: forktest.o $(ULIB)
        # forktest has less library code linked in - needs to be small
        # in order to be able to max out the proc table.
        $(LD) $(LDFLAGS) -N -e main -Ttext <b> 0x3000 </b> -o _forktest forktest.o ulib.o usys.o
        $(OBJDUMP) -S _forktest > forktest.asm

    ```
After unmapping pages, if we access the null pointer we will get trap 14 which indicates page fault as shown below
![Alt text](<xv6-public/Screenshots/project4/Pointer Dereference/after_trap.png>)

We will handle this trap in second part, after handling these it looks like this
![Alt text](<xv6-public/Screenshots/project4/Pointer Dereference/After_handling_PF.png>)


## Stack Rearrangement
we have to arrangement the existing architecture to new architecture as given below

USERTOP = 640KB                                  <br/>
stack (at end of address space; grows backwards) <br/>
... (gap >= 5 pages)                            <br/>
heap (grows towards the high-end)               <br/>
code                                            <br/>
(3 unmapped pages)                              <br/>
ADDR = 0x0                                      <br/>


In stack rearranging we have to modify all the below files
1. proc.h : To keep stack top in track for every process we added an variable called   `stacktop` in PCB

2. exec.c : After loading the user program into memory. we are allocating the memory for 156 pages because USERTOP has to be 640 KB means we have total of 160 pages. starting three pages are unmapped and one page is allocated for user code.So, we are allocating memory from sz to 156 * PGSIZE. To leave gap of atleast 5 pages we are clearing the page table entries using  `clearpteu`

3. vm.c : Firstly, In this file we are modifying the code in copyuvm, while copying we have to copy data starts from 3rd pages and till guard. After that we have to copy stack to child memory. After every copy we are mapping virtual address to physical address<br/>
secondly, we added a function call named `pagefault`, this is called while handling the page fault in trap.c. In this function first we are checking whether the current accessing address(addr) is valid or not, if it valid we allocate memory to stack. Further, we are checking whether the memory is allocated or not by verfying `stack_size`. Code snippet of this logic is as below

    ```
    if (addr >= guard && addr < stack)
    {
        int stack_pages = (stack - addr) / PGSIZE;
        int stack_size = stack - PGSIZE * (1 + stack_pages);

        stack_size = allocuvm(curproc->pgdir, stack_size, stack_size + PGSIZE);
        cprintf("Allocated memory starts at %d ends at %d\n",stack_size,stack_size + PGSIZE);
        if (stack_size == 0)
        {
        cprintf("Access is Invalid!\n");
        curproc->killed = 1;
        }
        else
        {
        lcr3(V2P(curproc->pgdir));
        cprintf("Allocate a new page!\n");
        }
    }
    else
    {
        cprintf("Incorrect access!\n");
        myproc()->killed = 1;
    }
    ```

4. trap.c : In trap.c we are calling pagefault function when a trap is raised with number 14 this is attained by writting a case statement in switch-case in trap method.

5. proc.c : In fork function after a creation of child PCB, parent stacktop is copy to child's PCB

6. defs.c : As we changed the number of arguments in copyuvm we have to modify here in defs.c and pagefault method declaration as well.

## Outputs for testing all previous project commands

### PROJECT 1 - Uniq, head and ps
Command 1: uniquser OS611_example.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/uniq_head/uniquser.png>)

Command 2:uniquser -c OS611_example.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/uniq_head/uniquser-c.png>)

Command 3:<br/>
        uniq OS611_example.txt<br/>
        uniq -c OS611_example.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/uniq_head/uniqkernel.png>)

Command 4:<br/>
headuser OS611_example.txt<br/>
headuser -n 3 OS611_example.txt OS611_example_1.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/uniq_head/headuser.png>)

Command 5:<br/>
head OS611_example.txt <br/>
head -n 3 OS611_example.txt OS611_example_1.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/uniq_head/headkernel.png>)

Command 6: ps
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/uniq_head/ps.png>)

### PROJECT 2 - Printing stats

Command 1 : test uniq OS611_example.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/test/test_uniq.png>)

Command 2 : test uniq -c OS611_example.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/test/test_uniq_option.png>)

Command 3 : test head OS611_example.txt OS611_example_1.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/test/test_head_mul.png>)

Command 4 : test head -n 3 OS611_example.txt OS611_example_1.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/test/test_head-n_mul.png>)

### PROJECT 3 - Scheduler

Command 1 : scheduler FCFS head j.txt # uniq j.txt # uniquser j.txt # headuser j.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/Scheduler/FCFS.png>)

Command 2 : scheduler FCFS headuser j.txt # uniquser j.txt # head j.txt # uniq j.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/Scheduler/FCFS1.png>)

Command 3 : scheduler PRIO head j.txt # uniq j.txt # headuser j.txt # uniquser j.txt
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/Scheduler/PRIO1.png>)

Command 4 : scheduler PRIO -o head j.txt 4 # uniq j.txt 1 # headuser j.txt 3 # uniquser j.txt 2
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/Scheduler/PRIO_number.png>)

Command 5 : scheduler PRIO -o head -n 3 j.txt 4 # uniq -d j.txt 7 # headuser -n 2 j.txt 6 # uniquser j.txt 5
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/Scheduler/PRIO_with_option.png>)

All the commands used in previous project are working as expected.

## Testing Criteria
1. From nlpointer.c program, it's clear that after accessing a address from first cases page fault. This confirms that pages are unmapped and assigned to NULL

2. Stack is growing after the memory allocated it is exceeded. This is verified using `test_recursion.c` program.
![Alt text](<xv6-public/Screenshots/project4/stack Rearrangement/stack_growth_test.png>)

3. Between stack and heap we allocated 5 extra pages and a guard pages to prevent over flow of stack

## resources

1. https://pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf
<br/>(After reading this PDF which is related to this topic gave me a big picture of existing architecture)

2. https://www.youtube.com/watch?v=67Q7tfUk6pI
3. https://www.youtube.com/watch?v=8V2YkO7lfvs&t=868s

    (This videos helped to relate conception knowledge to implmentation)




## Commands
```
uniquser OS611_example.txt
uniquser -c OS611_example.txt
uniq OS611_example.txt
uniq -c OS611_example.txt
headuser OS611_example.txt
headuser -n 3 OS611_example.txt OS611_example_1.txt
head OS611_example.txt
head -n 3 OS611_example.txt OS611_example_1.txt
ps


test uniq OS611_example.txt
test uniq -c OS611_example.txt
test head OS611_example.txt OS611_example_1.txt
test head -n 3 OS611_example.txt OS611_example_1.txt

scheduler FCFS head j.txt # uniq j.txt # uniquser j.txt # headuser j.txt
scheduler FCFS headuser j.txt # uniquser j.txt # head j.txt # uniq j.txt
scheduler PRIO head j.txt # uniq j.txt # headuser j.txt # uniquser j.txt
scheduler PRIO -o head j.txt 4 # uniq j.txt 1 # headuser j.txt 3 # uniquser j.txt 2
scheduler PRIO -o head -n 3 j.txt 4 # uniq -d j.txt 7 # headuser -n 2 j.txt 6 # uniquser j.txt 5

```

