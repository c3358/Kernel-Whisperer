# Kernel Whisperer

## What is it?

Kernel Whisperer encompasses three components:
* Kernel module: captures events and creates internal strings with enough information to identify the sources of those events. The events are stored internally using a queue implemented using a linked list.
* User-mode application: user-mode application that fetches event strings from the aforementioned queue and stores them on a database.
* Database (SQL, Non-SQL, NewSQL, Graph): database where the kernel events are stored. 

I have developed this tool as a means to collect kernel events in a way that could allow for mining, analytics, etc. 

## Which events are being caught?
* Filesystem:
	* Created Files
* Network:
	* TCP: outbound/inbound connect/data 
	* UDP outbound/inbound data
	* ICMP outbound/inbound data
* Process creation
* Registry:
	* Create Key (RegCreateKey, RegCreateKeyEx)
	* Set Value (RegSetValue, RegSetValueEx)
	* Query Key metadata (RegQueryInfoKey) 
	* Query Value (RegQueryValue, RegQueryValueEx)
	* Open Key (RegOpenKey, RegOpenKeyEx)
	
More events will be added to Kernel Whisperer as needed.

## How are the tables organized?
* The database is called Events.
* There are four tables:
	* File: ID, Timestamp, Hostname, PPid, PImageFilePath, Pid, ImageFilePath, Type (FILE_CREATED, FILE_OPENED, FILE_OVERWRITTEN, FILE_SUPERSEDED, FILE_EXISTS, FILE_DOES_NOT_EXIST), File
	* Registry: ID, Timestamp, Hostname, PPid, PImageFilePath, Pid, ImageFilePath, Type (CREATEKEY, OPENKEY, QUERYKEY, QUERYVALUE, SETVALUE, OPENKEY), RegKey, Value, Data
	* Network: ID, Timestamp, Hostname, PPid, PImageFilePath, Pid, ImageFilePath, Protocol (TCP, UDP, ICMP, UNKNOWN), Type (CONNECT, RECV/ACCEPT, LISTEN), LocalIP, LocalPort, RemoteIP, RemotePort,
	* Process: ID, Timestamp, Hostname, PPid, PImageFilePath, Pid, ImageFilePath, CommandLine

I am using the tuple ID+Timestamp to identify a record since KeQuerySystemTime does not provide a timestamp with enough precision to avoid collisions (i.e. events on the same table with the same timestamp).

**Note:** Currently, Kernel Whisperer will drop any database called Events before starting the insertions. You can tune this behavior by adjusting the queries on Client->lib->SQLDriver (header and source file).

## Interaction with the database
I am not leveraging any API to interact with the database. I wanted Kernel Whisperer to be versatile so that i could switch to any database with a couple of adjustments. As such, i have created a simple Python server that reads queries and executes them directly on the OS through the command line. 


## Ports, configurations and modules
* By default, the Python server runs on port 5000 and the client will interact with that same port. The port and IP address for the database can be changed on Client->lib->SQLDriver.


## How to compile and run?
1. Disable Windows driver integrity check and/or enable test signatures. This depends on the operating system. Starting on Windows 7 x64 you need to sign the driver using a Certificate generated by you. For older versions, it is enough to disable integrity verification. Details of the process will be omitted (Google is your friend here).
2. Install:
  * WDK 7.1.0 (https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit)
  * Visual Studio or the developper tools
3. Compiling and deploying driver:
   1. Start->Windows Driver Kits->WDK [VERSION]->Build Environments->Windows 7->x64 Checked Build Environment
   2. Change directory to the folder where Kernel Whisperer is located
   3. Run CompileAndDeployDriver.cmd. **NOTE:** This script will generate the certificate (amongst other stuff) and save it on Driver->staging. Once you generate the certificate you can comment the lines responsible for generating it.
4. Compiling and deploying client:
   1. Start->Microsoft Visual Studio [VS VERSION]->Visual Studio Tools->Developer Command Prompt for VS[VS VERSION]
   2. Same as before
   3. Run CompileAndDeployClient.cmd. 


## Test environment

Kernel Whisperer has been tested with the following environment:

* Windows 7 Enterprise SP1 x64 (client + driver) + MySQL(Ver 14.14 Distrib 5.5.59) on Linux remnux 3.13.0-53-generic
