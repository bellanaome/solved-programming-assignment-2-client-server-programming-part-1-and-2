Download Link: https://assignmentchef.com/product/solved-programming-assignment-2-client-server-programming-part-1-and-2
<br>
<h1>OVERVIEW</h1>

In this project you will implement a series of Client/Server programs on the Unix/Linux platform. The Client/Server model consists of a service provider (=server) and a requester of services (=client). Since the server and client(s) will run as independent processes, there are some operating system mechanisms necessary to provide communication between server and client(s). In this project, you will use two different inter-process communication primitives: a) unnamed pipes and b) named pipes (aka. Fifo’s). Unnamed pipes are covered in class. More information about named pipes can be found in the second part of this assignment.

We will use two different methods for implementing the server process:

<ul>

 <li><strong>Sequential server process:</strong> The requests are processed sequentially in a loop. Upon receiving a request, the request from the client, the server processes the request. Upon completion of the request, the server goes back to the beginning of the loop to handle the next request.</li>

 <li><strong>Concurrent server process: </strong>When a client request is made, the server process forks (system call: fork() ) off a child process and delegates the request to it. After completion of the request, the child process will terminate. The concurrent server can handle a specified maximum amount of simultaneous client connections (i.e., processes; this is called the <em>degree of concurrency</em>).</li>

</ul>

In addition, you will implement client-side code to establish a connection with the server and request some server side processing (see details below).

<strong> </strong>

<h1>PART 1: Simple “Hello World” Single Client/Server Application (Unnamed Pipe)</h1>

In the first part, you will implement a simple “Hello World” client/server application. All the functionality should be in a single source file: in other words, a single program creates the client process and the server process. The program should be named “cs_1” and does not need any additional command arguments.

Your program will first use the Unix pipe() system call to set a unidirectional communication primitive (called unamed pipe; see lecture slides for more details an examples). Subsequently, the program will use the fork system call to create a new process. After fork(), there will be two processes: the original process (the parent) and the new process (the child). The parent will take on the role as the server process, the child will be the client.

The parent process will simply execute an infinite loop in which it makes a read() system call on the read end of the unnamed pipe (see lecture slides) and wait for the client to ‘connect’. The client will prompt the user to input a command (i.e., just prompt the user to input a line of text). We consider 2 cases:

<ol>

 <li>The line starts with a “send:”. In that case the text following the colon will be send to the server. The server will receive the string (as a result of the read() system call on the read end side of the pipe and print a message on the screen: “server (PID:XXXX): received string &lt;client string&gt; from client (PID: YYYY)”. Subsequently, the server should loop back and execute another read() on the read end of the pipe. The “XXXX” and “YYYY” represent, respectively, the process id’s (PID’s) for the server and client (<em>Note: each process can obtain it’s own PID through the getpid() system call; the server receives the PID of the client as a result of the fork() system call</em>).</li>

 <li>The line starts with a “exit”. In that case the client will send an exit command to the server and the client should then exit. The server, once it receives the exit message, should print out the following message: “server (PID:XXXX) exits” and the exit() (Note: Each process needs to exit using the exit() or _exit() calls).</li>

</ol>

The format of the message to be send from the client to the server is as follows. Use the following C-structure:

#define MSG_LENGTH 100 // maximum length of message

typedef enum {   REGULAR,

COMMAND  } msg_type_t;

typedef struct msg {   msg_type_t type;

char message_text[MSG_LENGTH];

}msg_t;




Each message sent by the client to the server should be put in an instance of the above defined structure “msg_t”. The message type should be used by the server to identify whether a command (right now only the “exit” command) is sent (in this case the

“message_text” buffer would be empty, or an actual message to be printed. The actual sending can be done using the “write()” system call ( on the client side) and the “read()” system call (on the server side).

<h1>DELIVERABLES</h1>

Turn in your program with a makefile. Make sure the program is named “cl_1” as shown above. Submit your program using the “/home/drews/bin/442submit 3” option (or “/home/drews/bin/442submit -f 3 (if you have to overwrite a previous submission)).

<h1>GRADING</h1>

Total points: 20

Breakdown:

<ul>

 <li>Code is commented (2 points)</li>

 <li>Messages are sent using the “msg_t” structure as described above (5 points)</li>

 <li>The “send” command is correctly implemented as described above (10 points)</li>

 <li>The “exit” command is correctly implemented (3 points)</li>

</ul>

I will test the program on pu1.cs.ohio.edu. Should the makefile give me any errors and the program is not built, the score for the project will be 0 points.

<strong>PART 2 </strong>

In this part you will modify your part 1 solution in the following ways:




<h1>• [Modification 1: Create the client and server in two different programs:]</h1>

Create two separate programs: one for the client (called “cs_2_client”) and one for the server (called “cs_2_server)




<ul>

 <li><strong>[Modification 2: Use named pipes (Fifo’s):]</strong> Use named pipes (aka. Fifo’s) instead of unnamed pipes. Notice that while unnamed pipes are covered in class, named pipes (Fifo’s) are not. They are, however, somewhat similar The difference between them is that unnamed pipes are created through a system call called “pipe()”, and they require some sort of parent/child relationship between processes to be used. This is not true for named pipes. Named pipes are created through the “mkfifo()” system call and they are represented as a (pseudo) file in the file system. This allows any two processes on the same computer to communicate with each other (i.e., not only do they not have to be in a parent/child relationship, but they could even belong to processes created by different users!</li>

</ul>




Your server should create two named pipes (either in your own directory, or under /tmp). One named pipe will be used for reading (writing) by the server (client) and one for writing (reading) by the server (client). This is necessary since they represent a unidirectional communication channel. Make sure you only assign permissions to read/write the named pipes you’re creating to yourself (the owner of the pipes). More details on how to set up the pipes are given below.




<ul>

 <li><strong>[Modification 3: Use a ‘known’ server named pipe for any client to initially connect and use fork() in the server to create a server child process for each connecting client:] </strong>On the server side (i.e., in the “cs_2_server” program), when the server is first started, it will create a named pipe called “server_np”. This named pipe will serve as a known name (in the file system) for any clients to connect to the server. A client will have to perform the following steps to connect to the server:</li>

</ul>




<ol>

 <li>First, the client will create a pair of named pipes in the project directory (I would recommend you keep all the files in one directory for this implementation). These named pipes should be called “&lt;PID&gt;_send” and “&lt;PID&gt;_receive”, where “&lt;PID&gt;” represents the unique process ID (PID) of the client process. The purpose of this pair of pipes is for the later communication with the server client (see details) below). Two pipes are necessary because a single pipe can only be used for unidirectional communication and it will be necessary to have bidirectional communication between the client and the server (again, see below for more details).</li>

</ol>




<ol>

 <li>The client will then create a server request message (see definition of this structure below). This request message will contain the process ID of the client. The client will then and attempting to write this request message to the known server named pipe (“server np”). The server will basically execute an infinite loop, and in each loop iteration it will execute a blocking</li>

</ol>

“read()” on the server pipe (“server_np”). Whenever a client writes a message to the “server_np” pipe, the server will wake up from the read call and obtain the client request. Upon receiving this request, it will extract the

PID from the request message and then “fork()” to create a server child (“worker”) process. The server child is now responsible for carrying out the requests of the client, and communicating with it. This communication will be done using the “private” pair of named pipes the client established (see step a)).




Note that the communication structure as described above will require 2n+1 named pipes for a set of  n clients. This is because there will always be the one “server_pipe”, and each of the clients will need a pair of named pipes.




<ul>

 <li><strong>[Modification 4: Extend the commands for the communication between server and client(s):]</strong> Make the following changes to the communication protocol between server and client:</li>

</ul>




<ol>

 <li>The initial request by the client (i.e., the one to “server_np” will only require a request structure containing the client’s PID to be sent to the server (parent).</li>

</ol>




<ol>

 <li>Once the initial request has been received by the server (parent), the server will “fork()” an create a server (child). This server child will process all the remaining requests by the client. Note that unlike in Part 1 of the assignment, this remaining communication is through the named pipes “&lt;PID&gt;_send” and “&lt;PID&gt;_receive”. Also, in contrast to Part 1, the client and server will now perform bidirectional communication (i.e., the client sends a request through “&lt;PID&gt;_send” and receives an answer from the server through “&lt;PID&gt;_receive”.</li>

</ol>




In addition to the “exit” and “send:” command (see Part 1 of the assignment for details), implement a “status” and a “time” command:




<ol>

 <li>The “status” command. The “status” command will return some basic server statistics to the client. Namely, the server should return the current total number of clients being processes (i.e., the number of child processes the server parent has created and that have no exited yet) (see below for more details).</li>

</ol>




<ol>

 <li>The “time” command should return the time of day to the client (see below for more details).</li>

</ol>




<ul>

 <li>The “send:” and “exit” commands should behave just as in Part 1.</li>

</ul>




<strong>DETAILS: </strong>

<strong>Modification 2: </strong>

I will post some examples on how to use named pipes. This is a link to a good (short) tutorial on named pipes:




<ul>

 <li>Linux Journal article on named pipes: <a href="https://www.linuxjournal.com/article/2156">http://www.linuxjournal.com/article/2156</a> <strong>Modifications 3 and 4: </strong></li>

</ul>

You should use the “msg_t” structure that I gave you in Part 1 of the project and extend it appropriately. More specifically, you should create (at least) two structures (something like “msg_initial_request_t” and “msg_request_t”). One message should be used to describe the initial request by the client (Note: this message may be as simple as a structure with a single integer member, which represents the PID of the requesting client). The other structure should be used for the subsequent communication between the clients and the forked server child processes. You should start with the “msg_t” structure from Part 1 and extend it to support the additional commands “time” and “status”. (Note: the grade for this part of the project will depend on a good/reasonable design).

The “time” command will require the server (child) to obtain the time of day. Check out the following link for examples on how to do this in C/C++:

<ul>

 <li>http://www.catb.org/esr/time-programming/</li>

</ul>

The “status” command will require the server (child) to return the number of clients currently being processes to the server. This may seem conceptually simple at first, but it is a little bit more complicated than it seems. To simplify things, I only require the server (client) to return the total number of clients in the system right before the server parent forked the client. Thus, only the server (parent) needs to keep track of how many clients are being processed at the moment. But even that is not as simple as it seems. The server should maintain a variable (like “client_count”). Anytime a client is forked (that’s the easy part), this variable should be incremented. The problem is that the server (parent) doesn’t really know when a server (client) exits (i.e., upon receiving an “exit” command from the client).

In class we discussed signal handling. Signal handling can be used to keep track of the server (children) that exit the system. Anytime this happens, a signal (called SIGCHILD) is sent to the server (parent). You will need to define a signal handler for this signal. This signal handler should simply decrement the variable “client_count”) anytime it gets executed. See the lecture slides for more details on this.

<h1>DELIVERABLES</h1>

Turn in your program with a makefile. Make sure the programs built are named

“cs_2_client” and “cs_2_server”. None of them needs to implement any arguments.  Submit your program using the “/home/drews/bin/442submit 4” option (or

“/home/drews/bin/442submit -f 4 (if you have to overwrite a previous submission)).

<h1>GRADING</h1>

Total points: 65

Score Breakdown:

<ul>

 <li>Code is commented (5 points)</li>

 <li>Messages are sent using reasonably defined message structures as described above. More specifically, two message structures need to be created: one for the initial request to the server (parent), and one for all subsequent requests to the server (children) (10 points)</li>

 <li>The server properly accepts client’s initial requests and forks of server (child) processes. I will test your program with concurrent client sessions (i.e., I will open, for example, 4 terminal windows: one for the server, and three for three different clients. I will then attempt 3 concurrent sessions between clients and server). (20 points)</li>

 <li>The “send” command is correctly implemented as described above (5 points)</li>

 <li>The “exit” command is correctly implemented (5 points)</li>

 <li>The “time” command is correctly implemented (10 points).</li>

 <li>The “status” command is correctly implemented (10 points).</li>

</ul>

I will test the program on pu1.cs.ohio.edu. Should the makefile give me any errors and, the score for the project will be 0 points.








