Download Link: https://assignmentchef.com/product/solved-cs3210-lab-4-introduction-to-distributed-memory-programming-using-mpi
<br>
MPI (Message Passing Interface) is a standardized and portable programming toolkit for programming distributed-memory systems using message passing. It is supported by a few programming languages as an external library (there are many different implementations of MPI standard).

MPI Standards and Implementations

MPI (Message Passing Interface) is a <strong>message passing library standard </strong>based on the consensus of the MPI Forum which comprises of may commercial and research organizations and personnel. MPI Forum first released the MPI-1 standard in 1992 followed by MPI-2 (2000), MPI-2.1 (2008), MPI-2.2 (2009), and, MPI-3 (2012).

MPI-1 consisted of the basic standards required for message passing between nodes and the subsequent standards added more features.

Added in MPI-2: Dynamic Processes, One-Sided Communications including shared memory operations, Extended Collective Operations, External Interfaces to allows developers to add features such as debuggers, C++ Bindings, and, parallel I/O.

Added in MPI-3: Nonblocking Collective Operations (such as MPI_Iscatter), Neighborhood Collectives to be used with virtual topologies, Fortran Bindings, and, Performance Tools.

More information about the MPI standard can be found at MPI Forum documents here: <a href="https://www.mpi-forum.org/docs/">https://www. </a><a href="https://www.mpi-forum.org/docs/">mpi-forum.org/docs/</a><a href="https://www.mpi-forum.org/docs/">.</a>

Please do not confuse MPI standard with the MPI Implementations. The MPI standard defined by the MPI Forum as explained above is implemented by different organizations. Without these implementations, we cannot use MPI routines in our programs. Some of such MPI implementations include MPICH, IBM-MPI, and, OpenMPI, among others. In our labs we are using OpenMPI (<a href="https://www.open-mpi.org/">https://www.open-mpi.org/</a><a href="https://www.open-mpi.org/">)</a> which is an open source implementation of the MPI standard.

Computes in our lab have been installed with OpenMP 1.10.2 version with MPI-3 support. You can check these information by running ompi_info in the command line (terminal).

Part 1: Compiling and Running MPI Programs

An MPI program consists of multiple processes cooperating via series of MPI routine calls. Each MPI process has a unique identifier called rank. Ranks are by the programmer to specify the source and destination of messages as well as to conditionally control the program execution (ie. if rank=0 do this / if rank=1 do that). Processes are grouped into sets called communicators. By default, all MPI processes in a program belong to a communicator called MPI_COMM_WORLD.

Let’s look at the hello.c to identify the basic elements of an MPI program:

• Open the code in a terminal (console). You may use any available text editor:

&gt; vim hello.c

<ul>

 <li>To compile this program, use the command mpicc:</li>

</ul>

&gt; mpicc hello.c -o hello

<ul>

 <li>To run the program, use the command mpirun: &gt; mpirun -np 4 ./hello</li>

</ul>

mpirun is a script that receives at least two parameters:

<ol>

 <li>The number of processes to be created, -np &lt;n&gt;, where n is an integer.</li>

 <li>The path to the program binary. In this case, ./hello</li>

</ol>

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 1</u></strong>Run the program locally, the node where the program is downloaded, with 1, 4 and 32 processes and observe the output. Comment on the ordering of “hello world” messages.</td>

  </tr>

 </tbody>

</table>

<h1>Part 2: Mapping MPI Processes to Processors</h1>

Next, we compile another MPI program. We will run this program using two nodes called soctf-pdc-001 and soctf-pdc-002 (you need to replace the node names with the hostnames of the computers in your local cluster).

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 2</u></strong>Compile the program loc.c. If you compile the program on soctf-pdc-001 then run the program on the remote node soctf-pdc-002, or vice-versa.</td>

  </tr>

 </tbody>

</table>

• We can specify the node on which we need to run our MPI processes. In the below command, using -H flag directs MPI to run MPI processes on soctf-pdc-002 node.

&gt; mpirun -H soctf-pdc-002 -np 1 ./loc

Notice that mpirun asks for your password, and displays an error: ————————————————————————mpirun was unable to launch the specified application as it could not access or execute an executable:

Executable: ./loc

Node: soctf-pdc-002

while attempting to start process rank 0.

———————————————————————–This error occurs because the binary code is not found in the remote node, and you need to first copy the program binary to the remote host before execution.

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531">Note that in this command we use -np 1 flag. OpenMPI by default assumes that there is only one processing slot per node. Thus, if we try to a larger number of processes, we will get an error. (This does not happen when a machinefile is used as will learn in the next section.)</td>

  </tr>

  <tr>

   <td width="70"> </td>

   <td width="531"> </td>

  </tr>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 3</u></strong>Using the instructions from Tutorial 1, setup your SSH keys to connect to the remote host without typing your password.</td>

  </tr>

  <tr>

   <td width="70"> </td>

   <td width="531"> </td>

  </tr>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 4</u></strong>Copy the loc binary to the remote host and run the program.</td>

  </tr>

 </tbody>

</table>

The program outputs the location of each process that is started together with the processor affinity mask. MPI allows us to specify exactly where we want the processes to execute. The easiest way to accomplish this is to use a machine configuration file or machinefile. (machinefile is also referred to as hostfile in some documentation) This file contains a list of hostnames of the nodes where your processes will run. For example, you can create a file machinefile.1 as follows:

soctf-pdc-001

soctf-pdc-002

specifies that process 0 starts on node soctf-pdc-001 and process 1 on soctf-pdc-002. If there are more than two processes, mpirun will cycle through the listed nodes in a round-robin fashion by default. However, round robin can be scheduled by-slot or by-node (a slot is one processor on the node).

• Execute loc with mpirun while specifying the relevant machinefile with by-slot round

robin scheduling. If not specified explicitly OpenMPI assumes that the scheduling is by-slot

&gt; mpirun -machinefile machinefile.1 -np 10 –map-by slot ./loc | sort

This command will generate the following output (sort is used to sort the output text in alphabetical order):

Process 0 is on hostname soctf-pdc-002 on processor mask 0xff

Process 1 is on hostname soctf-pdc-002 on processor mask 0xff

Process 2 is on hostname soctf-pdc-002 on processor mask 0xff

Process 3 is on hostname soctf-pdc-002 on processor mask 0xff Process 4 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 5 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 6 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 7 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 8 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 9 is on hostname soctf-pdc-001 on processor mask 0xffff

• Execute loc with mpirun while specifying the relevant machinefile with by-node round robin

scheduling.

&gt; mpirun -machinefile machinefile.1 -np 10 –map-by node ./loc | sort

This command will generate the following output:

Process 0 is on hostname soctf-pdc-002 on processor mask 0xff

Process 1 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 2 is on hostname soctf-pdc-002 on processor mask 0xff

Process 3 is on hostname soctf-pdc-001 on processor mask 0xffff Process 4 is on hostname soctf-pdc-002 on processor mask 0xff Process 5 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 6 is on hostname soctf-pdc-002 on processor mask 0xff

Process 7 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 8 is on hostname soctf-pdc-001 on processor mask 0xffff

Process 9 is on hostname soctf-pdc-001 on processor mask 0xffff

Compare the above two examples and understand the difference in by-slot and by-node round robin scheduling.

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531">Note that if -np parameter is higher than the total number of slots on all nodes, you will get an error. However, you can force OpenMPI to overload using ”overload-allowed” directive. Read more about this and process placement here: <a href="https://github.com/open-mpi/ompi/wiki/ProcessPlacement">https://github.com/open-mpi/ompi/ </a><a href="https://github.com/open-mpi/ompi/wiki/ProcessPlacement">wiki/ProcessPlacement</a></td>

  </tr>

 </tbody>

</table>

A more precise control of the mapping can be achieved using a machine file and a rank file. The rank file lists each MPI rank number, starting with 0 and tells exactly which nodes, which sockets and which cores can be used by that rank (for MPI jobs, the process’ “rank” refers to its rank in MPI_COMM_WORLD). The rank file lists each rank on a line, with the following syntax:

 &gt; rank &lt;number&gt;=&lt;hostname&gt; slot=&lt;socket_range&gt;:&lt;core_range&gt;

For example, download the sample rank file rankfile.1 and take a look (shown below).

rank 0=soctf-pdc-001 slot=0:0 rank 1=soctf-pdc-002 slot=0:0 rank 2=soctf-pdc-001 slot=0:0

rank 3=soctf-pdc-002 slot=0:0-2 rank 4=soctf-pdc-002 slot=0:0-2 rank 5=soctf-pdc-002 slot=0:0-2 specifies that process 0 is mapped to node soctf-pdc-001 on core 0 of socket 0, then process 1 is mapped to node soctf-pdc-002 socket 0 core 0, process 2 on node soctf-pdc-001 socket 0 core 0 and then processes 3, 4 and 5 will be started on node soctf-pdc-001 on socket 0, and are free to be executed on either of cores 0, 1 or 2.

Here is an example of using rankfile.1

&gt; mpirun -machinefile machinefile.1 -rankfile rankfile.1 -np 6 ./loc

This command generates the following output:

Process 2 is on hostname soctf-pdc-001 on processor mask 0x1

Process 1 is on hostname soctf-pdc-002 on processor mask 0x1

Process 5 is on hostname soctf-pdc-002 on processor mask 0x7

Process 3 is on hostname soctf-pdc-002 on processor mask 0x7

Process 0 is on hostname soctf-pdc-001 on processor mask 0x1

Process 4 is on hostname soctf-pdc-002 on processor mask 0x7

For more details on mapping between processes and cores, consult mpirun’s manual. More information and further reading:



<ul>

 <li>LLNL MPI Tutorial: <a href="https://computing.llnl.gov/tutorials/mpi/">https://computing.llnl.gov/tutorials/mpi/</a></li>

 <li>OpenMPI FAQ: <a href="https://www.open-mpi.org/faq/?category=running">https://www.open-mpi.org/faq/?category=running</a></li>

 <li>Another MPI Tutorial: <a href="http://mpitutorial.com/">http://mpitutorial.com/</a></li>

</ul>

<table width="601">

 <tbody>

  <tr>

   <td width="60"></td>

   <td colspan="2" width="541"><strong><u>Exercise 5</u></strong>We have a total of 18 cores in your workbench (Xeon:10, i7:4 and Jetson:4). Run the program loc with 18 processes such that it maps each process to one core. Show your TA once you get it working.</td>

  </tr>

  <tr>

   <td colspan="2" width="70"></td>

   <td width="531"><strong><u>Exercise 6</u></strong>Download and compile the MPI matrix multiplication program, mm-mpi.c. Create and test the following four MPI configurations:1.    Using 4 processes, all on the Core i7 node, binding each process to one core.2.    Using 4 processes, all on the Core i7 node, without process binding.3.    Using 10 processes, all on the Core Xeon node, without process binding.4.    Using 18 processes, on all three nodes, without process binding.Explain what type of data distribution is currently used in the matrix multiplication problem. Explain advantages and disadvantages to this distribution. You may use performance measurements to prove your points. Change the implementation to use another data distribution type. Try to choose a distribution that might translate into a better or (roughly) similar performance (speedup).</td>

  </tr>

  <tr>

   <td width="60"></td>

   <td width="10"></td>

   <td width="531"></td>

  </tr>

 </tbody>

</table>

<h1>Part 3: Process-to-process Communication</h1>

MPI provides two types of process-to-process communication: blocking and non-blocking. Each of these communication types is accomplished with a send and a receive function.

<h2>Part 3.1: Blocking Communication</h2>

Blocking send stops the caller until the message has been copied over to the underlying communication buffer (i.e. network buffer). Similarly, blocking receive blocks the calling process until the message has been received in the MPI process.

The syntax for blocking send is:

————————————————————————int MPI_Send(void *buf, int count, MPI_Datatype datatype, int dest,

int tag, MPI_Comm comm)

————————————————————————-

The syntax for blocking receive is:

————————————————————————int MPI_Recv(void *buf, int count, MPI_Datatype datatype,

int source, int tag, MPI_Comm comm, MPI_Status *status)

————————————————————————Table 1 lists the arguments to be used with blocking communication functions.

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 7</u></strong>Compile the program block_comm.c and run it with two processes.</td>

  </tr>

 </tbody>

</table>

Table 1: Arguments for Blocking Communication Routines

<table width="640">

 <tbody>

  <tr>

   <td width="88"><strong>buf</strong></td>

   <td width="553"> </td>

  </tr>

  <tr>

   <td width="88"><strong>count</strong></td>

   <td width="553">The number of items that will be send.</td>

  </tr>

  <tr>

   <td width="88"><strong>datatype</strong></td>

   <td width="553">Specifies the primitive data type of the individual item sent in the message, and can be one of the following:MPI_CHARMPI_SHORT MPI_INTMPI_LONGMPI_UNSIGNED_CHARMPI_UNSIGNED_SHORTMPI_UNSIGNED_LONGMPI_UNSIGNED MPI_FLOATMPI_DOUBLEMPI_LONG_DOUBLEMPI_BYTEMPI_PACKED</td>

  </tr>

  <tr>

   <td width="88"><strong>dest/source</strong></td>

   <td width="553">Specifies the rank of the source / destination process</td>

  </tr>

  <tr>

   <td width="88"><strong>tag</strong></td>

   <td width="553">An integer that allows the receiving process to distinguish a message from a sequence of messages originating from the same sender</td>

  </tr>

  <tr>

   <td width="88"><strong>comm</strong></td>

   <td width="553">The MPI communicator</td>

  </tr>

  <tr>

   <td width="88"><strong>status</strong></td>

   <td width="553">Pointer to an MPI_STATUS structure that allows us to check if the receive has been successful.</td>

  </tr>

 </tbody>

</table>

Pointer to the memory buffer that holds the contents of the message to be sent or received

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 8</u></strong>Modify the file block_comm.c (new name block_comm_1.c) such that process 1 sends back to process 0 ten floating point values using only one message. Compile the program and run it.</td>

  </tr>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 9</u></strong>What happens if we flip the order of MPI_Send and MPI_Recv in the master process?Discuss the implication.</td>

  </tr>

 </tbody>

</table>

<h2>Part 3.2: Non-blocking Communication</h2>

Non-blocking communication, in contrast to blocking communication, does not block either the sender or the receiver.

The syntax for non-blocking send is:

————————————————————————int MPI_Isend(void *buf, int count, MPI_Datatype datatype, int dest,

int tag, MPI_Comm comm, MPI_Request *request)

————————————————————————-

The syntax for non-blocking receive is: ————————————————————————int MPI_Irecv(void *buf, int count, MPI_Datatype datatype,

int source, int tag, MPI_Comm comm, MPI_Request *request)

————————————————————————The only new parameter in the call is:

<table width="611">

 <tbody>

  <tr>

   <td width="61"><strong>request</strong></td>

   <td width="550">Using this handle, the programmer can inquire later whether the communication has completed.</td>

  </tr>

 </tbody>

</table>

Pointer of type MPI_Request which provides a handle to this operation.

Both functions return immediately, and the communication will be performed asynchronously w.r.t. the rest of the computation. Using these functions, the MPI program can overlap communication with computation. It is unsafe to modify the application buffer (your variable) until you know for a fact the requested non-blocking operation was actually performed by the library.

To check whether the functions have finished communicating, we can use:

————————————————————————-

int MPI_Test(MPI_Request *request, int *flag, MPI_Status *status)

————————————————————————MPI_Test takes a handle of an MPI_Isend or MPI_Irecv and store a true/false in the flag variable which indicate whether the operation has been completed or not.

————————————————————————-

int MPI_Wait(MPI_Request *request, MPI_Status *status)

————————————————————————MPI_Wait blocks the current process until the request has finished.

MPI provides quite a few variants of these basic two functions. You are encouraged to read the MPI manual for these functions:

————————————————————————MPI_Test, MPI_Testall, MPI_Testany, MPI_Testsome

MPI_Wait, MPI_Waitany, MPI_Waitsome

————————————————————————-

<table width="601">

 <tbody>

  <tr>

   <td width="61"></td>

   <td width="540">MPI does NOT guarantee fairness, thus programmer should make sure that operation starvation does not occur.</td>

  </tr>

  <tr>

   <td width="61"></td>

   <td width="540"><strong><u>Exercise 10</u></strong>Compile the program nblock_comm.c and run it with 3 processes. Modify the source (new name nblock_comm_1.c) code to flip the order of the MPI_Isend and MPI_Irecv. Compile and run it again. What do you observe?</td>

  </tr>

 </tbody>

</table>


