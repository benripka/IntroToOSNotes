<h1>Introduction Material</h1>

<h2>What Does the OS Do?</h2>

The operating system is used as an interface to the hardware, separating the computers specific hardware architecture from the details of application running on it. It allows for much more standardization and portability of code. It help manage the computer resources in a **fair** adn **optimized** way, often between multiple programs ( more technically processes). such hardware devices include I/O, memory, CPU, and more. through **CPU virtualization** adn **memory virtualization** any piece of software have behave under the assumption of infinite memory / processing resources. This is huge, and results in isolated address spaces. 

The OS is also able to facilitate communication between applications. By exposing a formalized API (called POSIX in UNIX systems) applications are able to take advantage of these *system calls* to make awesome software.

<h2>OS Breakdown: What's in it?</h2>

The UNIX type OS's can be broken down into three main categories:

<h3>Kernel</h3>
    The OS kernel is the root of the OS. It is loaded by the firmware boot loader when you turn on a machine and always executes in supervisor mode. It is the part of UNIX that makes a given operating system a UNIX types.
<h3>Daemons</h3>
    Operating systems also ship with man programs that are not part of the kernel but interact with it a lot on a very low level. These are called **daemons**. the first one to load is called systemd, which serves as the root and launches many other daemons thereafter.
<h3>Middleware</h3>
    Finally, there is middleware, which provide additional support between a users software and other peripheral devices such as databases, graphics, etc. In general daemons operate just on the core parts of the machine, like memory, CPU, storage, etc. For extra devices middleware is used.

<h2>The OS Hardware Interface</h2>

<h3>CPU types</h3>
CPU's can be single-processor (one cpu with registers, caches and a main memory), multiprocessor (many processors with there own registers and caches, but a shared main memory), or Multi-core (where there are multiple cpu's that have their own registers and L1 cache but share an L2 cache. They are created on the same chip). Multi-core is awesome because since the cpu's are not the same chip they can communicate with each other MUCH faster and with a lot less power costs.

<h3>Multi-programming</h3>

Multi-programming is when you load multiple processes into memory and then switch them in/out of the CPU when they either finish or need to wait for I/O. this increases the **utilization** of the CPU. A subcategory of this is called multitasking which essentially does the same but in a more complex way. Sometimes switching between them, just to ensure the other ones make progress, even if there was no I/O needed.

When the process asks for I/O, it is removed from the CPU and then enters the waiting queue of the device that it wants it from. When it gets served after waiting its turn, the data goes to a buffer and the device it came from sends an I/O interrupt to the CPU. This in turn tells the OS to put the process back into the ready queue, to wait its turn to hop back into the CPU, this time with a buffer full of the data that it wanted!

interrupt are handles by the CPU. They are electrical signals sent directly to the CPU, and cause it to immediately stop whatever its doing, and starts the interrupt handler routine, which basically looks the interrupt up in a table and execute the appropriate routine for the interrupt. Before doing so it has to load its state (before the interrupt) into a cache so it can properly resume execution as if nothing happened later. Then it does just that. The CPU actually has two pins for interupts. One is maskable (almost all device interrupts go here) and can therefore be ignored momentarily if the CPU is in a critical operation. The other is not maskable and cannot be ignored.

**Trap** interrupts are created by software running on the CPU. They can either represent a semantic error in execution (division by 0) or a System call. If the system call is made it means the software requests an OS service and therefore the OS takes over. Hardware cam further support software by making it secure. This is done by keeping a physical status bit for the operating mode. It will be in user mode unless the kerel runs a system call, in which case it has full priveleges. This is a reason to isolate some heavy operations as system calls.

<h2>System Calls</h2>

Only the OS can run these functions, and they occur in a priveleged way on the CPU. There is a few hundres of them. there are system calls for managing processes, allocating memory, file manipulation, and communication with othre processes aswell as alot more. 


<h2>Hardware Support</h2>
As dicussed above, there is alot of hardware support for things like security, but there is also support for address space virtualization / protection. The base and limit registers of a CPU hold the addresses of the top and bottom of a processes memeory space. The limit actually just holds the displacement from the base of the top. These are of course the references from which the physical addresses can be mapped to virtual addresses. The CPU can make sure that all references stay within this partition of the memory and through errors if not. therefore protecting the integrity of the memory.

The hardware has a timer that enforces periodic inturrupt that preven the CPU from being hogged incase there is no I/O. Its a form of multitasking.

<h2>Running  a Program</h2>

<h3>Compiling: g++ -c main.cc</h3>
This converts the source ascii file (just characters) into an object file. Software called the compiler does this. The object files are almost executable, but they are missing some things like function definitions that were included from other libraries, etc. This is solved through linking. The object files are written in assembly code (which is usually shown in hex)

<h3>Linking: g++ -o main main.o otherlib.o</h3>

This will generate an executable file that has all of the needed executable code, in the right order (if done right). Statically linked files are object files that are combined together before execution. DLL or dynamically linked files are only link if needed, at run time. This saves space at teh const (potentially) of efficiency.

<h3>Loading: ./main</h3>
The ./ operator is what loads a given executable file into the CPU. It calles the execv() system call to do so. Executable files are actually called ELF files. This means Executable and Linkable Files. They contain the main executable block aswell as metadata such as a symbol table. After loading the program turns into a process, which is like a live program. The ELF file has the text section - data section - and symbal map that contains the address to the starting point of where in the executable each function name starts. helpful for stack tracing and other debugger support.

<h2>Unix OS Structure</h2>

The entire kernel in Unix systems is one gigantic binary executable file. This is called a monolithic structure and makes communcation with the kernel very fast and efficient, therefore system calls (which are the root of all programs) have very little over head. The other modules in the OS (non-core/kernel) are can be loaded at either boot time or runtime. This DOES NOT require the kernel to recompile. Thank Jesus. This is possible because each module talks to the other through unified interfaces. The root of the linux terminal shows the whole OS "project". Each subfolder has a purpose:

    include: public headers
    kernel: core kernel components (e.g., scheduler)
    arch: hardware-dependent code
    fs: file systems
    mm: memory management
    ipc: interprocess communication
    drivers: device drivers
    usr: user-space code
    lib: common libraries

Some OS designes such as mac os use something called a micro kernel that allows for some features like communication over ports to be called more easily from teh program.

<h2>The Process Abstraction</h2>

This is a super important abstraction for OS design. It represents a program that is loaded into memory, and has a set of resources allocated to it and only it. It has a PID and might encapsulate many threads. No process can ever write to the memory of another process. The memory space of a process is divided into:

<ol>
    <li>Text: contains the code of the executable programs</li>
    <li>Data: Contains the global data from the program</li>
    <li>Heap: Contains data that is dynamically allocated throughout at runtime by malloc() or new</li>
    <li>The Stack: Which contains local data, return values, and in general anything that is temporary</li>
</ol>

Processes can be single or multi threaded. Threads are another OS abstraction that represents a unit of sequential instruction, that all share the same memory (heap + data + text) but have there own stacks. Each thread has its of PC, stack, and registers.

<h3>Process Control Block</h3>

the PCB is a struct that holds all the details about the process. It has memebers that describe everything from the threads within it, to the address space. The OS handles the juggling of processes in the CPU by using the PCB's as handles to them. It will move them around from the ready queue (waiting for CPU time) to the devices queues (waiting for I/O) to the zombie queue. (when it is finished but the parent doesnt want it back yet). Oh yea, every process has a parent, where system d is the parent to all daemon. The process cycle is a really important thing to remember.

<h3>Process Lifecycle</h3>

The process goes through 5 main stages, it can new, ready, running, waiting, and and terminated. This can be seen as a finite state diagram with just a little tiny bit of thought.

<h3>Scheduler</h3>

The scheduler is the program that controls the ready queue data structures and is charged with deciding which should be the next process to go into the CPU. It can be implemente with varous scheduling processes depending on the implementation desired. The scheduler is called very often and runs very fast (<10ms). **Context switching** is the process of going from one process to another before it finishs. This is often needed, but is computationally a heavy process, and is completly overhead. The entire state of the cpu is stored within the PCB of the process for later repopulation into the CPU. The scheduler chooses when this should occur and what process should take the next place, but hte **dispatcher** is what actually makes it happen! It does all the PCB loading and user mode switching, etc. Very low level stuff. Possibly the lowest level program haha. 


<h2>Process Management System Calls</h2>

There are many system calls that can be employed to manage thte execution of processes. Some of them include:

• getpid( ) returns the current PID
• fork( ) copies the current process (a new PID is assigned to the child process)
• exec( ) loads a new binary file into memory (without changing its PID)
• wait( ) waits until one of its child processes terminates
• waitpid( ) waits until the specified child process terminates
• exit( ) terminates a process
• kill( ) sends a signal (interrupt-like notification) to another process
• pause( ) causes the calling process to sleep until a signal is delivered that either
terminates the process or causes the invocation of a signal-catching function
• nanosleep( ) suspends execution of a process for at least the specified time
(can be interrupted by a signal that triggers the invocation of a handler)
• sigaction( ) sets handlers for signals

When a process is created through the use of the fork() call, everything is copied over, exept the program counter is reset to 0. Process termination is the ultimate resource reclamation step. It free's all resources and the process can either die or become a zombie. System calls can exit the process naturally or send abort signal or kill signals to do so more agressively.

Processes can have their priorities set, which can change the way in which the scheduler chooses it in the queue. The Nicer a process is the lower its priority. You can think of this as - the scheduler is ok hanging outo nice processes but wants to send the mean ones away. Selfish Schedular, poor CPU. You can also use the pthrac system call to allow one process to take contorl of the other for debuggin perposes. Or you can put the process to sleep with the sleep() command.

Shell systems are simply process management tools. You get a syntax to control the creation and organization of processes. This is truely what it means to use a computer. All command are just processes (or direct system calls).

<h2>Signals</h2>
Singles are a form of IPC (inter process communication) that consist of tiny bits of info. Somtimes they can carry a little data payload aswell. Signals should be created with the sigaction call and associated struct to improve he portability. handlers are set up to catch different signals, and a mask can also be setup to block some signals. The signal disposition and mask are copied into the child on fork(), and only removed then on exec().

<h2>Interprocess Communication</h2>

Process can either be cooperating or not. Cooperating processes work together to achieve a certain task / goal whereas others do not. IPC is used by cooperating tasks mostly. They can be used to exploit paralellism. UNIX allows for full duplex pipes to be setup between processes. There are a few forms of IPC. One is message passing. This requires kernel  intervention on every message read / wite. Heavy for streaming applications, but good because low error.An alternative is shared memeory which only needs to be setup once and then is very fast, but error prone.

<h2>Message Passing</h2>

In message passing, the kernel puts messages in a structure in the kernel, and the processes need to know where to read it.  There are many many ways to setup message passing. Simplex / Dubplex, Datagram / byte stream model, connection-oriented vs. connectionless. Message passing can be done in a blocking, non blocking, or partially blocking methods. Blocking means that if you send you wait for confirmation that it was recieved, and the reciever wait to get the message. Nonblocking means the sender can continue after sending and hte reciever doesnt have to be there right away, so the kernel has to hang onto it for a while. In the parial case, you just set an uperbound (timeout) on the time waiting on the other pid.

To setup shared memory, simply map some memory for it and then fork the process, therefore the child will no exactly where the memory is! Smart! There are more advanced system calls to handle the allocation of specifically shared memory address spaces.

<h1>Pipes</h1>

Unnamed pipes can only be used to communicate between parent and children parents. Named pipes can be used by all. Reading is always done through the first part of the pipe, and writing always through the other. Pipes are simply files and when created return a file descript, or two actually, one for each end. FIFO is used to implement Named pipes. They are duplex, and much more powerful. They are heavier implementations in the unix Kernel than others.

<h1>Distributed Systems</h1>

A distributed system is a set of loosly coupled nodes, connected by a communication network. This can take the form of electrical signal busses, or the internet in general. Each **node** has its own resources such as memory, cpu, devices etc. and have there **own operating systems** therefore communication accross the network can be thought of as OS to OS communication. Nodes in a DS communicate through message passing, in relationships such as peer to peer, client server model. By using DSs you can improve speed, reliablity and minimize the amount of resources in the system by increasing resource sharing throughout it.

<h2>Challenges of Design</h2>

You need to make a robust DS. One that can withstand the loss or degredation of a message. One that can detect hardware failures anywhere in the system. You might use a heartbeat protocol in this case to do frequent all systes checks. The system **shouldn't seem any different to the user**. This is called transparency to the user. Further, the system should react to increases in load by scalling up resources, and maintain that all the resources (files) are in sync with eachother if duplicated.

There is a difference between a **Network OS** and a **Distributed OS**. The first is just an OS that enables the user or users software to communicate with other OS's on the network by establishing a session and communicating via system calls. In this case the user is AWARE of the seperation between machines/OS's. Distributed Operating Systems are really a differnet beast though. The API they expose is slightly higher level in a way, hiding the complexity of the file / process transfer between machines of the network, and presenting to the user / programmer all resources as if they were simply on the same machine. They can make it very easy to migrate data from place to place, but also to send bits of computation (threads) from place to place in a way that minimizes network trafic and execution time, thus power and time. 

<h2>Basics of Communication</h2>

- A **NIC** is a network interface card and presents a hardware coded network interface address that make the machine unique in the system.
- A **Packet** is the small unit of sequential bits that comprise a message, and are sent of over the network in one go. Its just a sequence of bits, thats all.
- A **protocol** is simply the rules that all systems conform to regarding communication. The how-to's of the network.

<h2>Network Types</h2>
  
- **LAN** is the local area network and is a small wifi network or ethernet network of like a building. Very small and every node in it gets all packets, no matter what. They simply discard them if they are not the recipient though. WIFI is the name of the protocol for this communication.
- The **WAN**'s are the wide area network and can span entire countries or the whole world. They communicate from large node to node over 'links' which is a general word for large connection that can take on many medium forms, like telephone lines, leased (dedicated data) lines, optical cable, microwave links, radio waves, and satellite channel. Speeds can vary in WAN's and they are inherently less reliable than LAN's.

WAN's and LAN's can certainly interconnect, and do very often. Messages are chopped up into packets are launched accross the network, their flow controlled by the nodes in the network. When a packet arrives at a node, an interupt is created so that the OS will handle it right away. Routers are used in the WAN's to find where a packet should be delivered. It reads the packet header and launches it on to the next node closest to the destination. They all keep lookup tables to remember what is where. You can uniquely identify the destination with a (host-name, pid) couple. The hostname is found in the DNS. It is the domain name system. These are IP addresses, or the more human readable form that looks like a URL.

<h2>TCP / IP Stack</h2>

This provides some abstractions that allow for the creation of software and thought at different levels of complexity. Starting from the base you have:

- The **link layer** which deals mainly with frames. Frames treat packets as fixed sized things are handle the complexity of interfacing with the actual transport medium.
- The **Internet Layer** deals with routing the packets properly from machine to machine in the network. Thats why you call it an IP adress. Its the internet protocol address, which is a property of this layer of thinking. You need to identify, encode and decode things on this layer using IP addresses.
- The **Transport layer** is concerned with creating packets, and handling the order in which they are transported. It is where you decide whether to use TCP or UDP, and partition the message into these packets and arrange their order accordingly. You make decisions about how you'd like to transport data. Ports and port numbers belong to this layer. Ports allow the machine with a single IP to have each process use its own port. By specifying the port a message should go to and wait att you can specify the process that it will go to. Some ports are allocated for just the system, others can be used by any software running on the machine, but always only one at a time.
- The **Application Layer** is the highest level and allows processes to work with an API that hides everything and lets it communicate directly with other processes, even on machines accross the world. It has all the complexity of everything below it. 

the Medium Access Control address is the actual number that is exposed and uniquly identifies a machine in the global network. This is used on the Link layer, and the computer keep a mapping from know IP addresses ot MAC addresses.

TCP is a form of standard implemented on the transport layer, and it is fault tolorant and connection oriented. The connection is created when both sides know which port on the other machine they should be communicating with. Setting up a session through TCP involves a 3 step hadnshake. SYN, SYN + ACK, ACK. Therefore they both send the synchronization data and both know that the other got it properly. The Transport layer also has implementations in place to help congestion and flow control.

The **socket** is an abstraction of the network I/O queue, where messages are put and lined up to be ready to the transport layer to break them up and the other layers do their thing. There are system calls in unix to allow software to write to sockets. Sockets can either be **stream sockets** or **datagram sockets**. The former is reliables, adn ensures that all data gets there. The latter is just a best effort, not connectoin oriented and just shoots it off, disregarding potential flaws in the network. its like TCP vs. UDP. the socket() system call creates one, and return the file descriptor to it. After that you can pretty much treat it asa a file, and ensure you close it afterwards. You can duplicate it, write to it, and read from it. With the sockets you must specify the IP hostname of the machine and the port that the process will be running at. When you create a socket, you can then bind() the socket to a port, and therefore make it ready to accept connections. You can also get the port name from a socket that is already bound with getsockname().Finally you should call listen() on the bound socket, therefore telling it to passively wait for connections. create with socket() => connect to port with bind() => wait for connections with listen() => accept(). The accept call creates a new socket that will persists after the connection is established. once this is all set up accept() will block and wait for someonne ot try and connect. The client will connect() with that sys call after socket() => bind() => connect(). Once the connection is setup you can freely use the recvfrom() and sendto() system calls to do jus that! The listen() => accept() and connect() phases are not needed for UDP sockets. Finnally after communicating you must call the close () syscall on the socket fd. This will terminate the connection.

<h1>Scheduling</h1>

You can define a process as CPU bound, IO bound, or a mix of the two. Thi smeans it spends more or less time waiting to either one of the two. If you are running IO bound processes the pqueue will be almost empty all the time.

- **workload** is a the tasks the system must do. 
- **performance metrics** are ways of telling is a given scheduling algorithm is good or bad for a given workload. Throughput, reponse time, etc.
- **scheduling policy** is the decision about which types of processes to execute first. 
- **ocerhead** is the extra non-useful work done by the scheduler. its imporant because the scheduler runs very often.
- **fairness** tells what fraction of the CPU cycles goes to a given task.

There is long term scheduling, which chooses what process to load into memory and how many (degree of multiprogramming) to begin with. short-term scheduling is the spu scheduling that chooses what runs out of the ready queue at a given moment. Scheduling is often done online, or on the fly, but if the bust lengths are know and execution order you can actually get better performance by doing it offline. The scheduler runs when a process ends, in the cpu, when an interupt occurs, and when a process is created (it needs to find where it should go in the ready queue). A more complex scheduler might be **preemptivde** which basically means its able to cause its own interrupts on the CPU to switch things up. Some of the performance metrics are given:
  
-  CPU Utilization (higher is better)
 percentage of time CPU is busy
- Throughput (higher is better)
 number of processes completing in a unit of time (e.g., in one second)
 overhead reduces the throughput
- Waiting time (lower is better)
  total amount of time that process is in ready queue
- Turnaround time (lower is better)
 length of time it takes to run process from initialization to termination, including all waiting time
- Response time (lower is better)
 what user sees from keypress to character on screen
 time between when process is ready to run and its next I/O request (or completion time)

You need to choose an algorithm that will best suit a given appplciation. They are not all the same, infact its very hard to find a way to optimize them all. the following examples of algorithms that assume there is 1 CPU with 1 core and each process has only 1 thread (very simplified):

- **FCFS** or first come first serve is the base. It just completely finished tasks in order of recieving them. FCFS will still however kick out a process if it requests IO and throw it at the back. You can make simple calculations for these to get the average turn around time (time from entering the queue to completion for each task) and the waiting time which is how long its in the ready queue. It might help to draw a diagram where you see the contents of the CPU and the flow of time as things finish. Fairly straight forward in this case. You could also create a table specifying the burst time, and arrival time of each. Lots of the metrics are expressed as averages. the table might be handy if the arrival times are not all at 0. Recall that turn around and wait times are from THAT PROCESSES arrival time, and may differ from the others. Its overall not that good because it has extremly variable wait times and leaves IO devices idle.
- **Round Robin** scheduling involves setting a timer to premptively pop the process out of the CPU if it takes to long. each pid gets a max time to try and finish. Either to quick or too slow is bad. For equal size tasks this isnt great. It can improve turnaround alot thogh. Stops huge tasks from blocking the shit out of tiny little things. 
- **SJF** is the shortest job first. It will use estimates of burst time to pick the shortest one and thus take advantagde of I/O wait times. Problem is long running CPU ones can get igored. You can estimate the burst lenght by using a moving average estimator.

Starvation meeans that a process will never be picked to run in the CPU. One of th best ways to optimize the process is with multilevel priority queues, were priority is assigned by the user through nice(). The priority queues each run a round robin with exponentially increasing deltas until the bottom which is straight up FCFS. The high priority ones need to be done at high delta so they all have the chance to get there. Large jobs will move to lower queues and therefore won't block the other stuff. Its a natural filer. This setup is called a MFQ, or multilevel feedback queue. Very smart.

<h1>Threads</h1>

A thread is a single sequence of execution that can be treated as a seperatly scheduled task. Threads have their own stack for local variables / temportaries like return calues and aprameteres. They share the heap but often can have special memory called thread local memory aswell. All the details of a thread are stored in the TCB (thread control block). The TCB has the tid, scheduling info (niceness) aswell as much more. They are an improvement on multi process systems for cooperating tasks. That's because ALL of their memory is shared, so no need for like, message passing or any of that shit.

<h2>Parallelism</h2>

Concurency and Paralellism are NOT the same thing. You can have concurancy without parallellism. Concurency just means that all process make progress, not just one after another. context switching (scheduling) can do this aswell as parallelism. In order to use parallelism you must identify the divisions between tasks. Tasks should be independant of eachother. NExt, you need to split up the data in a way that stops conflict between the operation of the threads. If there is any data dependancies they must be synchronized properly. The OS has its own threads called Kernel threads to exploit paralellism for itself. This helps becayse context switching is much cheapter between threads than processes. User level threads are not recognized or scheduled by the OS. All that work is done explicitly by the programmer, using creation, deletion, synchronization, yield functions given by libraries. In user level threads the schduling algorithm is defined in a problem dependant way IN the software. It can do this by having a thread give up the processor to others. This is called yielding. User level threads are usually much faster than kernel level threads. 

User level threads cause a problem though, because the scheduler is unaware of them. It treat to processes the same even if one has way way way more threads, giving the same time and even pauseing it entirely if just a single thread makes an IO call. Not good. Usually implementations involve jsut creating  a kernel thread for every user level thread. this forms a 1 to 1 relation that is handles by the thread library (pthreads for example). These will handle the complexity of setting up the threads. 

<h2>POSIX Threading</h2>

The complexity of mapping to kernel threads is usually hidden by library functions (pthreads) or by the compiler itself (demarkated by #pragma parallel). Thread creation can work in eiher the user domain or kernel domain. It is implemented by the pthread library in both for unnix systems, and WIN 32 for the others. Some of the pthread calls can be seen below:

- pthread_create( ) creates a separate thread and returns its handle
‣ takes a start_routine which is invoked with the specified arguments
- pthread_attr_init( ) sets the initial attributes of a thread
‣ scheduling information, stack size, stack address, etc.
- pthread_join( ) allows the calling thread to wait for the specified thread to
terminate
‣ the calling thread is blocked until that currently executing thread has completed
- pthread_yield( ) causes the calling thread to relinquish the CPU voluntarily
- pthread_exit( ) terminates the calling thread and executes clean-up handlers
defined by pthread_cleanup_push( )


Threads can be setup to handle threads where all threads recieve async signals, or just the first to listen. Threads can independantly choose to mask signals after creation. Sync singals will be sent to the threads that created it only. You can send a signal to a specific thread if you know it by going pthread_kill() with it. You can cancel a thread in two way, let it die when its ready or just kill it righ away, purhaps losing resources as you do. A thread can also set its own cancelation points that exist for healthy cancelation. Many blocking calls are builtin cancelation points because the thread might hang there for a bit before being canceled.

Using threads to say... Implement a server, can be super shit because if you get a boat load of requests at once you run the risk of running out of resources. Threadpool cure this. You know what that is. The fork join model is basically the same as a threadpool but it runs sequentially. Threads share all global variables and thus synchronization primitives should be used to protect it. 

A **race condition** occurs when the order in which threads use shared data impacts the result of the thread execution. 

Dekkers algorithm is a solution to the milk prblem, it says you should set your lock, and then while the other threads lock is set aswell you check if its your turn. We can use this support to eliminate the need for things like busy waiting 

<h2>Memory Protection</h2>

The MMU is the memory management unit. It is used to translate beteween virtual addresses (that the CPU will read from the code in a process) and translates it into physical addresses. You can perform the translation at compile, load, and execution time. If you do it at compile time the physical and logical addresses must be the same. If you do it at execution time then you need hardware support from the **MMU**. In multiprogramming you need address protection so that they can all exist under the addumption of infinite address space. The cost of memory protextion hardware is simply two registers and a comparator circuit. The two registers will keep (for every process) the memory that its space starts at, and the size offset. The hardware checks every time for out of bounds read / writes and sends out a trap error if they are attempted. 

In **Static relocation** means before the process is executed, the OS goes through the memory and makes all the addresses physical. This can be done simply by adding to each of them the base registers contents (lowest address value allocated to the process). **Dynamic Relocation** translates addresses to physical on the fly. This incurs a small computational overhead but also allows the process to be moved around in RAM on the go. in DR, the base register is called the relocation register, and we need another adder circuit to find the PA on the fly.

**Fragmentation** can be internal or external to a process, and is simply unused address space. There are many differenct types of **allocation algorithms** that will impact the type of fragmentation that can occur. Policies are considered good if they reduce fragmentation. Worsdt-fit usually isn't that good. 
- **First Fit** allocation simply puts the process in the first block that is big enough. It is very simple but slow as you will potentially need to search the whole mem space. It needs to maintain a list of all blocks currently in the memory, sorted by address location. It can also cause external fragmentation as small processes die.
- **Best-fit allocation** is done by looking for the minimum sized free block that the process fits in. It minimizes fragmentatino but is slower to allocate / deallocate. It maintains a list of free blocks and merges if possible on deallocation, so it needs to check this list. 
- **Worst-fit** is used to minimize the amount of small fragments. It works best if there is medium sized allocations. 

**Memory compaction** is done by moving processes around to reduce the fragmentation in the memory. This can be done in many ways but it is most important that the processes don't notice the change. If there is not enough space in RAM for the processes, you can swap a process out and into the HD and put it back when it becomes active again. Compaction becomes much easier with swapping as you can settle things together one at a time.

<h2>Segmentation</h2>

So far we've considered a direct translation from a set of virtual adress $[1, MAX_{proc}]$ to physical address [1, MAX$_{sys}$]. This is great and manages to pull off the abstraction layer providing infinite isolated address space to each process, but sometimes we don't want process to be completly isolated! Sometimes we want them to be able to share some amount of data. In order to do this we need a way to section off data at a level that the programmer could potentially specify / work with. This is what segmentation does. Segments are groups of addresses use by a process. In segmentation MM, the virtual address created by the compiler consists of a segment number, and an offset from that segment. Segments are usually some form of division in the process that makes logical sense to the programmer, for example the stack might be in one segment, a function in another, etc.

The addresses can then be translated via a **segmentation table** which is a table where at the row given by a segment number you can find the base and limit adresses and therefore add the offset of the vitual address to them / Check that the address is within the limits for protection! The **problem** however with segmentation is that the segments are of ARBITRARY size, since they just depend on how big a funciton is, etc. So allocation becomes very challenging. Ontop of this you can't utilize memory very well, because the WHOLE segment needs to go into memory in order for it to work, but in reality you quite often do not use most of the memory in a segment. the **90/10** rule says that you spend 90% of the time accessing just 10% of a processes memory. 

<h2>Paging</h2>

Paging seeks to solve the same kind of problem as segmentation, but also make it such that the enitre memory space of a given process need not be loaded into memory for it to work. Instead of segments of variable length, the process virtual memory is partitioned into same sized blocks called pages. Pages are then translated to Frames of the same size, so virtual addresses consist of page numbers and offsets (that can be converted to frame numbers and offsets by looking at the page table). The nice thing is that you can actually take pages in and out of main memory as you please, swapping them out to secondary disk storage. The reason this is good is that the process will spend the vast majority of its time in just a few pages of the process. So you can use a smart allocation algorithms to deside when a page isn't being used, and therefore swap it out, making room in memory for other things! This is very important for multiprogramming machines, which are most modern machines. But the page table becomes very important and we want to be able to find the frame number very fast for a given page number. the page table is stored in memory aswell, which can be up to a 100ns. Instead, a small set of registers called the **Translation LookAside Buffer** of TLB which can be stored of chip in really fast chache. It is so small though that you cannot fit all the page numbers in it, so it only actually stores the common ones. The general policy is to look in the TLB no matter what, if you find the page num you're all good. If not, find the page num in memory (which takes alot more time), load it into the TLB and continue, so now you have it to use later! The TLB has a valid bit to show that the value stored there is relevant to the current process. This is because the stuff in there might be from an old processes logical space, which is different but may still have the same page numbers! Uh Oh!

If there was no TLB it would cost about $\mathrm{EAT}=2 \times C_{m a}$ seconds to get your memory where $C_{m a}$ is the cost of a single memory address. This is because you would need  to access the table (from memory) to find the frame num for a page, then access the memory at that frame num to legit get the entry! So what about with a TLB? Well first off there is only a probability that the page number will be in there and valid, so it may still take this double memory access time, but look at this:

$$\mathrm{E} \mathrm{AT}=\rho_{h i t}\left(C_{m a}+C_{T L B}\right)+\left(1-\rho_{h i t}\right)\left(2 \times C_{m a}+C_{T L B}\right)$$

Assuming there is a decent hit (page num was in TLB) probability, you usually only have to access the main memory once and the TLB once. If you had a miss, then there is SLIIIGHTLY more time then before but because of the 90/10 rule, usually the TLB will get near-optimally populated as the program runs! This is what the TLB is all about: Putting the page numbers that are common in a real accessible place. 

When you load a new process who's memory has k pages, you first load those k pages into k free frames of memory. If there isnt k free frames (because of the other processes) you make them free by disgarding stuff that isn't needed any more. While you do this you make sure to load the page numbers and their new corresponding frame nums into the in-memory page table. Finally, you sweep over the TLB and mark all entries as invalid (flush it) because anything in their is from the last process. Then the process can start, and as it goes in will poopulate and pseudo optimize its contents to match the pages that are frequently accessed. On a context switch, both the TLB and the page table values are stored in the processes PCB. Then, when it gets loaded back in they can be repolulated after flushing both and then it restarts asif nothing even happened!

Paging allows for a convenient optimization of the common fork() system call that duplicates an entrire process. What you can do on a fork is not copy any of the pages, but rather wait and see if the child actually tries to write to them. If it does, then copy over that page. By doing it on the fly you avoid the waste that would come as it calls exec() immediatly, which is common. 

<h2>Segmented Paging</h2>

We liked segmentation because it gave us the ability to break down the virtual memory space into logical segments that made sense to the programmer! Functions, stack, etc. But we really liked paging because it removes external fragmentation and lets us optimize the stuff we actually store in memory to the stuff we use often. Since pages are usually much smaller than segments, why not do both? Why not break the space into segments, then each segment be made up of pages, then each page translate to a fram? This is calleded segmented pages. In this case the VA consists of (segment number)(page number)(offset) and you can translate to PA by using the page number as an offset from the entry in a page-segment table given by the segment number index, and then that will givve you the frame number. Now you just add the offset to the frame num, do some range checks, and voi la. 

The problem is though, for all of these methods the address space can get VERY huge. We're talking like these page tables start to get bigger than a page itself, which will make thigns even more complex. So what do we do? One way of dealing with this is by using **hierarchical page tables** which essentially means not that the page number consists of its own internal page number and offset. This means you need multiple page tables so their no longer contiguous. This basically allocates extra memory for storing addresses in and of themselves. This starts to get rediculous for large address spaces like in the case of 64 bit architectures because each time you access a new page table (for each layer of the hierarchy) oyu need to do a whole memory access!! and holy shit, this starts to get expensive right away, anything above three should not be a thing...

A solution is the **hashed page table** the HPT takes in a page number and stores both that page number and its corresponding frame number in a table at the index given by the HASH of its page number. There may be multiple at one hash index so that's why we need to store the page number aswell, so they can be compared to find the actual one. The tradeoff here is achieving less memory accesses at the price of some extra pure computation (exectuing the hash function and collision correction logic [checking for the entry with the right page number if muliple]) 

<h2>Virual Memory Abstraction</h2>

So as we know the whole point of virtual addresses are protection from other processes and to give the programmer an illusion of infinite memory. This is only made possible however if we can run a process WITHOUT having all this potentially infinite memory loaded. This is made possible by swappiness with the disk, but we must ensure that the code that is put in memory is the code that will often be used! This of course prompts a new question, when should a page be loaded into memory at all? You could let the programmer decide, you could leave it to the compiler, but more likely its  a good idea to simply put the page into memory the first time that it is references, and evist the last page to be referenced. This strategy is called **demand paging** There is also such a thing as predictive paging, though the pursuit is difficult as there exist many different branches in the code. when the entire program is not in memory, the page tables valid bit is used to tell whether or not it is currently in memory. If not, a PAGE FAULT is called which causes teh kernel to take control and go to the DISK and retrieve the desired page. The OS uses a classic IO queue to make the process wait for access to the disk, it is then replaced in the ready queue when it is ready.

If a page consists of purely code, you don't actually need to store the page in the swap space on the harddrive when it is evicted, because you can always re read it from the executable file. so at any given moment the page couold live in three places! We'll need to rememeber where in the page file. Demand paging only really works because of the general tendency for programs to exihibit temporal and spatial locality. So basically, it is more likely that things will be accessed that are in the same page as the last instruction. This is why the following ETA calculation tends to result in faster access times:

$$\left(1-\rho_{\text {fault}}\right) C_{m a}+\rho_{\text {fault}} C_{\text {pagefault}}$$


