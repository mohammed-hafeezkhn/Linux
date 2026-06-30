* [Process](#process)
* [Threads](#threads)
* [Scheduling](#scheduling)

---

# Process 
- A process is a program (object code stored on some media) in the midst of execution.
- Processes are, however, more than just the executing program code (often called the text
section in Unix).They also include a set of resources such as open files and pending signals,
internal kernel data, processor state, a memory address space with one or more memory
mappings, one or more threads of execution, and a data section containing global variables.
- Processes, in effect, are the living result of running program code.The kernel needs to
manage all these details efficiently and transparently.

- A process is usually defined as an instance of a program in execution; thus,
if 16 users are running vi at once, there are 16 separate processes (although they
can share the same executable code). 

- Processes are often called tasks or threads

- kernel's point of view, the purpose of a process is to act as an entity to
which system resources (CPU time, memory, etc.) are allocated
- A process begins its life when, not surprisingly, it is created. In Linux, this occurs by
means of the fork() system call, which creates a new process by duplicating an existing
one.

<details>
  <summary><strong>What is a process?</strong></summary>

> - a program loaded into memory
> - identified by a process id, commonly
abbreviated as pid
> - owns resources:
    - memory, including
code and data
    - open files
    - identity - user id, group id
    - timers and more
> - Resources owned by one process are protected
from other processes
</details>

## Process Types
- The process that calls fork() is the parent, whereas the new process is the child.The
parent resumes execution and the child starts execution at the same place: where the call
to fork() returns.The fork() system call returns from the kernel twice: once in the parent process and again in the newborn child.
Often, immediately after a fork it is desirable to execute a new, different program.
- The
exec() family of function calls creates a new address space and loads a new program into
it. In contemporary Linux kernels, fork() is actually implemented via the clone() system call,
Finally, a program exits via the exit() system call.This function terminates the process
and frees all its resources.
- parent process can inquire about the status of a terminated
child via the wait4()1 system call, which enables a process to wait for the termination of
a specific process.When a process exits, it is placed into a special zombie state that represents terminated processes until the parent calls wait() or waitpid()

- When a process is created, it is almost identical to its parent. It receives a (logical)
copy of the parent's address space and executes the same code as the parent,
beginning at the next instruction following the process creation system call.
Although the parent and child may share the pages containing the program code
(text), they have separate copies of the data (stack and heap), so that changes by
the child to a memory location are invisible to the parent (and vice versa).

- a chronological thread (the computer follows a single path through the code, no other paths run at the same time), and a set of resources
allocated to the application — for example, memory, files, and so on. New processes are generated using
the fork and exec system calls:
  - fork generates an identical copy of the current process; this copy is known as a child process. All
resources of the original process are copied in a suitable way so that after the system call there
are two independent instances of the original process. These instances are not linked in any way
but have, for example, the same set of open files, the same working directory, the same data in
memory (each with its own copy of the data), and so on.3
  - exec replaces a running process with another application loaded from an executable binary file.
In other words, a new program is loaded. Because exec does not create a new process, an old
program must first be duplicated using fork, and then exec must be called to generate an additional application on the system.

### Process Descriptor
- To manage processes, the kernel must have a clear picture of what each process is
doing. It must know, for instance, the process's priority, whether it is running on a
CPU or blocked on an event, what address space has been assigned to it, which
files it is allowed to address, and so on

- This is the role of the process descriptor —
a task_struct type structure whose fields contain all the information related to a
single process

- The process descriptor contains the data that describes the executing program—open files, the process’s address space,
pending signals, the process’s state, and much more

- The task_struct is a relatively large data structure, at around 1.7 kilobytes on a 32-bit
machine.This size, however, is quite small considering that the structure contains all the
information that the kernel has and needs about a process

- As the repository of so much information, the process descriptor
is rather complex. In addition to a large number of fields containing process
attributes, the process descriptor contains several pointers to other data structures
that, in turn, contain pointers to other structure

- The kernel stores the list of processes in a circular doubly linked list called the task list.Each element in the task list is a process descriptor of the type struct task_struct.

- struct structure is allocated via the slab allocator to provide object reuse and
cache coloring

- For each process, Linux packs two
different data structures in a single per-process memory area: a small data structure
linked to the process descriptor, namely the thread_info structure, and the
Kernel Mode process stack. The length of this memory area is usually 8,192 bytes
(two page frames).

-  that a process in
Kernel Mode accesses a stack contained in the kernel data segment, which is
different from the stack used by the process in User Mode. Because kernel control
paths make little use of the stack, only a few thousand bytes of kernel stack are
required. Therefore, 8 KB is ample space for the stack and the thread_info
structure. However, when stack and thread_info structure are contained in a single page frame, the kernel uses a few additional stacks to avoid the overflows
caused by deeply nested interrupts and exceptions.



# Threads
- Threads of execution, often shortened to threads, are the objects of activity within the
process. 
- Each thread includes a unique program counter, process stack, and set of processor registers.
- The kernel schedules individual threads, not processes.To Linux, a thread is just a special kind of process
- threads share the virtual memory abstraction, whereas each
receives its own virtualized processor.
- user programs having many
relatively independent execution flows sharing a large portion of the application
data structures. In such systems, a process is composed of several user threads (or
simply threads), each of which represents an execution flow of the process.

- Linux uses lightweight processes to offer better support for multithreaded
applications. Basically, two lightweight processes may share some resources, like
the address space, the open files, and so on. Whenever one of them modifies a
shared resource, the other immediately sees the change. Of course, the two
processes must synchronize themselves when accessing the shared resource.

- A straightforward way to implement multithreaded applications is to associate a
lightweight process with each thread. In this way, the threads can access the same
set of application data structures by simply sharing the same memory address
space, the same set of open files, and so on; at the same time, each thread can be
scheduled independently by the kernel so that one may sleep while another remains
runnable. 
- POSIX-compliant multithreaded applications are best handled by kernels that
support "thread groups ." In Linux a thread group is basically a set of lightweight
processes that implement a multithreaded application and act as a whole with
regards to some system calls such as getpid( ) , kill( ) , and _exit( )


<details>
  <summary><strong>What is a thread?</strong></summary>

> - a thread is a single flow of execution or
control
> -  a thread has some attributes:
• priority
• scheduling algorithm
• register set
• CPU mask for SMP
• signal mask
• and others
> - all its attributes have to do with running code
> - Threads run in a process:
– a process must have at least one thread
– threads in a process share all the process
resources **Threads run code, processes own resources**
</details>

# Scheduling
Processes provide two virtualizations: a virtualized
processor and virtual memory
- The virtual processor gives the process the illusion that it
alone monopolizes the system, despite possibly sharing the processor among hundreds of other processes
- Virtual memory lets the process allocate and manage memory as if it alone owned all the memory in the system (<a href="Memory_Management.md"><kbd>&emsp;Memory Management&emsp;</kbd></a> 
<br><br>)

