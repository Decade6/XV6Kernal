# XV6Kernal
A series of modifications to the XV6 kernal

This project consists of a series of modifications/additions to the base XV6 Kernal.


**PSTree:**

The pstree program calls getprocs with an array of struct uproc objects and
sets max to the size of that array (in the unit of struct uproc objects). For
example, the program can malloc 64 struct uproc objects (assuming the returned
pointer is saved in pcs) and call getprocs with getprocs(64, pcs);. The kernel
stores up to max entries of the process information into the array, starting at
the first slot of the array and filling it consecutively. The kernel returns the
actual number of processes in existence at that point in time, or -1 if there was
an error. After receiving the process information from the kernel, pstree in
the user-space formats it into a tree structure.

First, the system call sys_getprocs was defined in the file sysproc.c
Then, the pstree.c program will utilize the getprocs system call.
I wrote the code for the sys_getprocs(void) function in sysproc.c and the main function in pstree.c.
This is very similar to the pstree utility in unix, but simplified.


**Priority Based Scheduler:**

Replaces the round-robin scheduler for xv6 with a priority-
based scheduler. The valid priority for a process is in the range of 0 to 200,
inclusive. The smaller value represents the higher priority. For example, a process
with a priority of 0 has the highest priority, while a process with a priority of
200 has the lowest priority. The default priority for a process is 50. A priority-
based scheduler always selects the process with the highest priority for execution.
If there are multiple processes with the same highest priority, the scheduler uses
round-robin to execute them in turn to avoid starvation. For example, if process A,
B, C, D, E have the priority of 30, 30, 30, 40, 50, respectively, the scheduler
should execute A, B, and C first in a round-robin fashion, then execute D, and
execute E at last.

For this part, I modified proc.h and proc.c. The change to proc.h is
simple: just add an integer field called priority to struct proc. The changes to
proc.c are more complicated. I first added a line of code in the allocproc
function to set the default priority for a process to 50. Xv6’s scheduler is
implemented in the scheduler function in proc.c. The scheduler function is called by
the mpmain function in main.c as the last step of initialization. This function will
never return. It loops forever to schedule the next available process for execution. I then replaced the
scheduler function with my implementation of a
priority-based scheduler. The major difference between my scheduler and the
original one lies in how the next process is selected. My scheduler loops through
all the processes to find a process with the highest priority (instead of locating
the next runnable process). If there are multiple processes with the same priority,
it schedules them in turn (round-robin)..

A solution to this problem is called aging. Specifically, if the process uses up its CPU time, I chose to
decrease its priority by 2 (i.e., add 2 to its priority since lower numbers represent
higher priority); if a process is waken up from waiting, increase its priority by 2.
In this part, I added code to function wakeup1 in proc.c, and function
trap in trap.c.
