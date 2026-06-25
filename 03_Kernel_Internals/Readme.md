# OS Internals

## 1. Overview & Architecture

## Linux Kernel Overview
### 1. The Core Role and Architecture of the Kernel
The kernel is the fundamental program of an operating system, loaded into RAM at boot. It acts as an intermediary between hardware and software, serving as an "enhanced machine" that abstracts hardware complexities, a resource manager for CPU and memory, and a secure execution environment for applications.

* **Execution Modes:** To protect system integrity, CPUs provide hardware-enforced privilege levels. Applications run in non-privileged User Mode, while the kernel runs in fully privileged Kernel Mode.
* **System Calls:** When a user application needs a hardware resource (like reading a file), it issues a system call. This triggers a temporary, secure switch to Kernel Mode so the OS can fulfill the request.
* **Kernel Architecture:**
  * **Microkernels** run only core functions in Kernel Mode and delegate the rest to external processes (highly modular but slower).
  * **Monolithic kernels** (traditional Unix) pack all subsystems into one large program running in Kernel Mode (very fast but rigid).
  * **The Linux Hybrid:** Linux is monolithic for performance but uses Loadable Modules. Modules (like device drivers) can be dynamically linked or unlinked at runtime, providing microkernel-like flexibility and memory efficiency without sacrificing speed.

### 2. Process and User Management
Operating systems are designed to be multiuser and multiprocessing, allowing multiple users and applications to run concurrently and securely.

* **Users and Groups:** System access is protected by authentication. Users are identified by UIDs (User IDs) and share files via GIDs (Group IDs). The root user (superuser) bypasses standard protections for system administration.
* **Processes:** A process is a running instance of a program. Linux uses preemptive scheduling, meaning the OS rapidly switches CPU time between active processes to create the illusion of simultaneous execution.
* **Creation:** Unix generates new processes using `fork` (duplicates the parent) and `exec` (loads a new program).
* **Threads:** Lightweight execution flows that run in parallel within the same process, sharing the same address space and data (created in Linux via `clone`).
* **Namespaces:** A feature that groups global resources so different processes have entirely isolated views of the system, forming the basis for modern containerization.

### 3. Memory Management
Because a system's virtual address space is usually much larger than its actual physical RAM, the kernel meticulously maps and manages memory.

* **Virtual vs. Physical Space:** Every process gets its own isolated virtual address space, divided into Userspace (accessible to the application) and Kernel Space (strictly reserved for the OS).
* **Page Tables:** The kernel maps virtual memory "pages" to physical memory "page frames." To manage the massive number of addresses efficiently, Linux uses a multilevel paging system, accelerated by hardware features like the MMU and TLB cache.
* **Physical Allocation (Buddy System & Slab Cache):**
  * The **Buddy System** allocates large, continuous blocks of memory in powers of two, automatically merging free blocks to prevent memory fragmentation.
  * The **Slab Allocator** builds on top of the buddy system to handle requests for tiny memory chunks, acting as a highly efficient cache for frequently used kernel objects.
* **Swapping:** To virtually expand RAM, the kernel temporarily moves inactive memory pages to a hard disk and retrieves them via "page faults" when the application needs them again.

### 4. Filesystems, Devices, and I/O
The traditional Unix philosophy states that "everything is a file." The kernel abstracts hardware interaction so applications can read/write to devices just like regular files.

* **Device Drivers:** Hardware is accessed via device files. Character devices (e.g., modems) are accessed sequentially byte-by-byte. Block devices (e.g., hard disks) allow random access and utilize extensive caching mechanisms.
* **Virtual Filesystem Switch (VFS):** An abstraction layer that allows the kernel and applications to interact uniformly with vastly different filesystems (Ext3, VFAT, XFS) using the exact same standard system calls.
* **Caching:** Linux heavily relies on the Page Cache to keep data recently read from slow block devices in fast RAM, significantly boosting overall system performance.

### 5. Unique Kernel Programming Mechanics
Developing the Linux kernel is drastically different from writing standard applications. Errors can crash the entire system, and developers must handle intense multi-processing concurrency, strict hardware requirements, and the lack of standard C libraries.

* **Linked Lists:** The kernel heavily utilizes cyclic, doubly linked lists (`list_head`) to dynamically chain data structures without relying on strict type-safety.
* **Kernel Objects (kobjects):** An object-oriented abstraction embedded within kernel structures. It enables reference counting (tracking how many subsystems are using an object to know when it is safe to delete) and groups objects to export system information to userspace via the `sysfs` filesystem.
* **Architecture Independence:** To run on diverse hardware (from 32-bit embedded chips to 64-bit mainframes), the kernel uses specialized data types (e.g., `__u8`), explicit byte-order conversions (Big Endian vs. Little Endian), and Per-CPU variables to optimize cache performance across multiple processors.


---

<br>

<a href="Memory_Management.md"><kbd>&emsp;2. Memory Management&emsp;</kbd></a> 
<br><br>
<a href="Process_Task_Management.md"><kbd>&emsp;3. Process Task Management&emsp;</kbd></a> 
<br><br>
<a href="IPC_and_Synchronization.md"><kbd>&emsp;4. IPC & Synchronization&emsp;</kbd></a> 
<br><br>
<a href="Interrupts_and_Exceptions.md"><kbd>&emsp;5. Interrupts and Exceptions&emsp;</kbd></a> 
<br><br>
<a href="Filesystem.md"><kbd>&emsp;6. Filesystem&emsp;</kbd></a>
<br><br>
<a href="Time_Management_and_Timers.md"><kbd>&emsp;7. Time Management & Timers&emsp;</kbd></a> 
<br><br>
<a href="IO_Control_and_Device_Drivers.md"><kbd>&emsp;8. I/O Control & Device Drivers&emsp;</kbd></a> 
<br><br>
<a href="Networking.md"><kbd>&emsp;9. Networking&emsp;</kbd></a>
<br><br>
<a href="Security_and_User_Control.md"><kbd>&emsp;10. Security & User Control&emsp;</kbd></a> 
<br><br>
<a href="Bootstrapping_and_Initialization.md"><kbd>&emsp;11. Bootstrapping & Initialization&emsp;</kbd></a> 
<br><br>
<a href="Virtualization_and_Resource_Isolation.md"><kbd>&emsp;12. Virtualization & Resource Isolation&emsp;</kbd></a>