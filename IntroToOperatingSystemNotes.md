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

• CPU Utilization (higher is better)
- percentage of time CPU is busy
• Throughput (higher is better)
- number of processes completing in a unit of time (e.g., in one second)
- overhead reduces the throughput
• Waiting time (lower is better)
- total amount of time that process is in ready queue
• Turnaround time (lower is better)
- length of time it takes to run process from initialization to termination, including all waiting time
• Response time (lower is better)
- what user sees from keypress to character on screen
- time between when process is ready to run and its next I/O request (or completion time)