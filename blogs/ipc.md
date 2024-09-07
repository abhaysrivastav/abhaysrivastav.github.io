# IPC
Inter-Process Communication (IPC) is the mechanism by which processes running in an operating system communicate with each other. The goal of IPC is to enable data sharing, synchronization, and signaling between processes to ensure they can cooperate effectively, even though each process typically has its own address space and execution context.

Process can be running on the same OS or might be running on different machines connected via a network. 

# Types of IPC Mechanims
## Pipes:
  - Most commonly used for process running on the same system.
  - Useful when process need to comuunicate or pass data sequentially.
  - One process sends the data(write) another process receive it (read).
  - 2 Types of pipes :
      - **Unnamed Pipes** : Used to communicate between a parent process and its child. These process are created in memory and do not have a name in file system.
      - **Named Pipes** : Also known as FIFO, extends the concept of Unnamed pipes, by giving the name in the file system. This allows communication between unrelated process.


### How Unnamed Pipes Work
An unnamed pipe is created using the **pipe()** system call in Unix based system. Data written to write -end of the pipe can be read from read-end by another process.If pipe read-end has no data available the reading process will block(wait). 

### How Named Pipes Works
A named pipe is created using system calls **(mkfifo(FIFO_FILE, 0666))**, once created any process can open named pipe for reading and writing. Data written at Write-End can read from Read-end.

  - Pipes are generally fast and efficient for small to medium-sized data transfers.
  - Pipes are designed for communication between processes on the same machine, not across a network.
  - Suitable for scenarios where you need to pass data sequentially between processes without requiring complex synchronization.
  - Effective for transferring data that doesn’t require extensive buffering or high performance.
  

## Message Queue
Allow process to communicate with each other by sending and receiveing messages. Pipe provides the byte streams, Message Queue allow process to exchange **discreate messages**.
Processes can send and receive messages independently of each other.  Some message queue implementaion allow priortization of messages, so higher priorty messages that are processed first.

### Basic Message Queue Operations
  - Creating a messaging queue
  - Sending a message
  - Receiving a message
  - Controlling the queue

Message Queue mechanism is provided by **System V Message Queue** or **POSIX IPC**  in Unix based OS.In windows  **Windows Messaging Queuing (MSMQ)** is provided.   

## Shared Memory 
In this a block of memory is allocated that can be accessed by multiple processess. Unlike other IPC methods, where data is copied between processess(e.g pipes or message queues) , shared memory allow processess to communicate by reading or writing directly to a shared memory segment. Its fastest method for IPC. 

### How Shared Memory Works
  - **Creation and Allocation**: A shared memory segment is created and a unique identifier is given in memory space. 
  - **Attachement** : Process that need to use the shared memory must attach it to their address space using the identifier.
  - **Access**: Like normal memory processess can read and write the attached shared memory address space.
  - **Detach**: When memory is not required to access process can detach it.
  - **Deletion** : When no process is accessing the shared memory it can be marked for deletion.

IN Unix based system it can be implemented using **System V shared Memory** or **POSIX**.In windows it can be implemented using **Memory Mapped Files**. 

  - Shared memeory is accessible to all process that attach to it so security is a concern.
  - Multiple processes can access the shared memory simultaneously, proper synchronization mechanisms (e.g., semaphores, mutexes) are necessary to prevent data corruption and ensure consistency.


## Semaphores 


## Sockets
- Sockets are low-level networking abstraction that allows communication between diffeent processess, which might be running on different systems over a network.
- Supports 2 way communication, that means data can be sent or receives.
- Socket can use different protocols like TCP or UDP.
### How Socket Works  
  - A socket is created using system calls by giving domain, protocols.
  - The socket is bound to an address
  - Socket is set to listen incomiing connections.
  - A client socket attempts to connect to server sockets.
  - Once connected , data can be exchanged between cleint and server.
    
### Different types of Sockets  
  - TCP Sockets
  - UDP Sockets
  - Raw Sockets

### Use Cases for sockets
  - Webserver and Client
  - Chat Applications
  - File Transfer
  - Network Monitoring tools

## Signals 
