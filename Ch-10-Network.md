As systems become more distributed, especially with cloud computing environments, the network plays a bigger role in performance. Common tasks in network performance include improving network latency and throughput, and eliminating latency outliers, which can be caused by dropped or delayed packets.

Network analysis spans hardware and software - 
- hardware is the physical network, which includes the network interface cards, switches, routers, and gateways (these typically have software, too)
- software is the kernel network stack including network device drivers, packet queues, and packet schedulers, and the implementation of network protocols. Lower-level protocols are typically kernel software (IP, TCP, UDP, etc.) and higher-level protocols are typically library or application software (e.g., HTTP)

# Terminology
**Interface**: The term interface port refers to the physical network connector. The term interface or link refers to the logical instance of a network interface port, as seen and configured by the OS. (Not all OS interfaces are backed by hardware: some are virtual.)

**Packet**: The term packet refers to a message in a packet-switched network, such as IP packets.

**Frame**: A physical network-level message, for example an Ethernet frame.

**Socket**: An API originating from BSD for network endpoints.

**Bandwidth**: The maximum rate of data transfer for the network type, usually measured in bits per second.

**Throughput**: The current data transfer rate between the network endpoints, measured in bits per second or bytes per second.

**Latency**: Network latency can refer to the time it takes for a message to make a round-trip between endpoints, or the time required to establish a connection (e.g., TCP handshake), excluding the data transfer time that follows.

# Models
## Network Interface
![Network Interfaces](./images/Ch10/Ch10-Network-Network-Interface.png)

Network Interface 
- is an operating system endpoint for network connections
- are mapped to physical network ports as part of their configuration. Ports connect to the network and typically have separate transmit and receive channels.

## Controller
![Controller](./images/Ch10/Ch10-Network-NIC-Controller.png)

provides one or more network ports for the system and houses a network controller: a microprocessor for transferring packets between the ports and the system I/O transport. An example controller with four ports is pictured

## Protocol Stack
![Protocol Stack](./images/Ch10/Ch10-Network-Network-Protocol-Stack.png)

# Concepts
## Networking and Routing
![Network connected via routers](./images/Ch10/Ch10-Network-Network-connected-via-routers.png)

Host A can connect to host B via the local network, usually driven by a network switch. Host A can connect to host C via router 1, and to host D via routers 1, 2, and 3. Since network components such as routers are shared, contention from other traffic (e.g., host C to host E) can hurt performance.

Connections between pairs of hosts involve unicast transmission. Multicast transmission allows a sender to transmit to multiple destinations simultaneously, which may span multiple networks. This must be supported by the router configuration to allow delivery. In public cloud environments it may be blocked.

## Protocols
Network protocol standards, such as those for IP, TCP, and UDP, are a necessary requirement for communication between systems and devices. 

Other protocols perform differently by design, and may be selected when they suit the workload: examples include *Stream Control Transmission Protocol* (SCTP), *Multipath TCP* (MPTCP), and *QUIC*

## Packet Size
- For TCP/IP and Ethernet, packets can be between 54 and 9,054 bytes, including the 54 bytes (or more, depending on options or version) of protocol headers.
- Packet size is usually limited by the network interface maximum transmission unit (MTU) size, which for many Ethernet networks is configured to be 1,500 bytes.
- Ethernet now supports larger packets (frames) of up to approximately 9,000 bytes, termed jumbo frames. These can improve network throughput performance, as well as the latency of data transfers, by requiring fewer packets.
- The performance of 1,500 MTU frames has been improved by network interface card features, including TCP offload and large segment offload. These send larger buffers to the network card, which can then split them into smaller frames using dedicated and optimized hardware. This has, to some degree, narrowed the gap between 1,500 and 9,000 MTU network performance.

## Latency
can be measured in different ways
-  name resolution latency
-  ping latency, connection latency
-  first-byte latency
-  round-trip time
-  connection life span

These are described as measured by a client connecting to a server

### Name Resolution Strategy
Establishing connections to remote hosts, a host name is usually resolved to an IP address, for example, by DNS resolution. The time this takes can be measured separately as name resolution latency.

### Ping Latency
This is the time for an ICMP echo request to echo response, as measured by the ping(1) command. This time is used to measure network latency between hosts, including hops in between, and is measured as the time needed for a network request to make a round-trip

### Connection Latency
Connection latency is the time to establish a network connection, before any data is transferred. For TCP connection latency, this is the TCP handshake time. Measured from the client, it is the time from sending the SYN to receiving the corresponding SYN-ACK. Connection latency might be better termed connection establishment latency to clearly differentiate it from connection life span.

Connection latency is similar to ping latency, although it exercises more kernel code to establish a connection and includes time to retransmit any dropped packets. The TCP SYN packet, in particular, can be dropped by the server if its backlog is full, causing the client to send a timer-based retransmit of the SYN. This occurs during the TCP handshake, so connection latency can include retransmission latency, adding one or more seconds.

Connection latency is followed by first-byte latency.

### First-Byte Latency
Also known as **time to first byte (TTFB)**, first-byte latency is the time from when the connection has been established to when the first byte of data is received. This includes the time for the remote host to accept a connection, schedule the thread that services it, and for that thread to execute and send the first byte.

While ping and connection latency measures the latency incurred by the network, first-byte latency includes the think time of the target server. This may include latency if the server is overloaded and needs time to process the request (e.g., TCP backlog) and to schedule the server (CPU scheduler latency).

### Round Trip Time
Round-trip time (RTT) describes the time needed for a network request to make a round trip between the endpoints. This includes the signal propagation time and the processing time at each network hop. The intended use is to determine the latency of the network, so ideally RTT is dominated by the time that the request and reply packets spend on the network (and not the time the remote host spends servicing the request). RTT for ICMP echo requests is often studied, as the remote host processing time is minimal.

### Connection Life Span
Connection life span is the time from when a network connection is established to when it is closed. Some protocols use a keep-alive strategy, extending the duration of connections so that future operations can use existing connections and avoid the overheads and latency of connection establishment (and TLS establishment).

## Buffering
TCP employs buffering, along with a sliding send window, to improve throughput. Network sockets also have buffers, and applications may also employ their own, to aggregate data before sending.

Buffering can also be performed by external network components, such as switches and routers, in an effort to improve their own throughput. Unfortunately, the use of large buffers on these components can lead to *bufferbloat*, where packets are queued for long intervals. This causes TCP congestion avoidance on the hosts, which throttles performance.

## Connection Backlog
TCP implements a backlog, where SYN requests can queue in the kernel before being accepted by the user-land process. When there are too many TCP connection requests for the process to accept in time, the backlog reaches a limit and SYN packets are dropped, to be later retransmitted by the client. The retransmission of these packets causes latency for the client connect time. The limit is tunable: it is a parameter of the *listen(2) syscall*, and the kernel may also provide system-wide limits.

Backlog drops and SYN retransmits are indicators of host overload.

## Congestion Avoidance
Networks are shared resources that can become congested when traffic loads are high. This can cause performance problems: for example, routers or switches may drop packets, causing latency-inducing TCP retransmits. Hosts can also become overwhelmed when receiving high packet rates, and may drop packets themselves.

There are many mechanisms to avoid these problems; these mechanisms should be studied, and tuned if necessary, to improve scalability under load. Examples for different protocols include:

- Ethernet: An overwhelmed host may send pause frames to a transmitter, requesting that they pause transmission (IEEE 802.3x). There are also priority classes and priority pause frames for each class.
- IP: Includes an Explicit Congestion Notification (ECN) field.
- TCP: Includes a congestion window, and various congestion control algorithms may be used.

## Local Connections
Connecting via IP to localhost is the IP sockets technique of *inter-process communication (IPC)*. Another technique is *Unix domain sockets (UDS)*, which create a file on the file system for communication. Performance may be better with UDS, as the kernel TCP/IP stack can be bypassed, skipping kernel code and the overheads of protocol packet encapsulation.

For *TCP/IP sockets*, the kernel may detect the localhost connection after the handshake, and then shortcut the TCP/IP stack for data transfers, improving performance. This was developed as a Linux kernel feature, called *TCP friends*, but was not merged [Corbet 12]. BPF can now be used on Linux for this purpose, as is done by the Cilium software for container networking performance and security [Cilium 20a].

# Architecture
## IP

## TCP
TCP can provide a high rate of throughput even on high-latency networks, by use of buffering and a sliding window. TCP also employs congestion control and a congestion window set by the sender, so that it can maintain a high but also reliable rate of transmission across different and varying networks. Congestion control avoids sending too many packets, which would cause congestion and a performance breakdown.

Important topics for TCP performance include the three-way handshake, duplicate ACK detection, congestion control algorithms, Nagle, delayed ACKs, SACK, and FACK.

### 3 way handshake
![TCP - 3 way handshake](./images/Ch10/Ch10-Three-Way-Handshake.png)

Connection latency from the client is indicated, which completes when the final ACK is sent. After that, data transfer may begin.

This figure shows best-case latency for a handshake. A packet may be dropped, adding latency as it is timed out and retransmitted.

Once the three-way handshake is complete, the TCP session is placed in the ESTABLISHED state.

### States and Timers
TCP sessions switch between TCP states based on packets and socket events. The states are LISTEN, SYN-SENT, SYN-RECEIVED, ESTABLISHED, FIN-WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT, and CLOSED [Postal 80]. Performance analysis typically focuses on those in the ESTABLISHED state, which are the active connections. Such connections may be transferring data, or idle awaiting the next 
event: a data transfer or close event.

A session that has fully closed enters the TIME_WAIT2 state so that late packets are not mis-associated with a new connection on the same ports. This can lead to a performance issue of port exhaustion.

Some states have timers associated with them. TIME_WAIT is typically two minutes (some kernels, such as the Windows kernel, allow it to be tuned). There may also be a “keep alive” timer on ESTABLISHED, set to a long duration (e.g., two hours), to trigger probe packets to check that the remote host is still alive.

### Duplicate ACK Detection
Duplicate ACK detection is used by the fast retransmit and fast recovery algorithms to quickly detect when a sent packet (or its ACK) has been lost. It works as follows:
- The sender sends a packet with sequence number 10.
- The receiver replies with an ACK for sequence number 11.
- The sender sends 11, 12, and 13.
- Packet 11 is dropped.
- The receiver replies to both 12 and 13 by sending an ACK for 11, which it is still expecting.
- The sender receives the duplicate ACKs for 11.

Duplicate ACK detection is also used by various congestion avoidance algorithms.

### Retransmits
Two commonly used mechanisms for TCP to detect and retransmits lost packets are:

- Timer-based retransmits: These occur when a time has passed and a packet acknowledgment has not yet been received. This time is the TCP retransmit timeout, calculated dynamically based on the connection *round-trip time (RTT)*. On Linux, this will be at least *200 ms (TCP_RTO_MIN)* for the first retransmit,3 and subsequent retransmits will be much slower, following an exponential backoff algorithm that doubles the timeout.

- Fast retransmits: When duplicate ACKs arrive, TCP can assume that a packet was dropped and retransmit it immediately.

To further improve performance, additional mechanisms have been developed to avoid the timer-based retransmit. One problem occurs is when the last transmitted packet is lost, and there are no subsequent packets to trigger duplicate ACK detection. (Consider the prior example with a loss on packet 13.) This is solved by Tail Loss Probe (TLP), which sends an additional packet (probe) after a short timeout on the last transmission to help detect packet loss

### Congestion Controls
Congestion control algorithms have been developed to maintain performance on congested networks. Some operating systems (including Linux-based) allow the algorithm to be selected as part of system tuning. These algorithms include:
- Reno
- Tahoe
- CUBIC
- BBR (Used by Netflix)
- DCTCP

The congestion control algorithm can make a large difference to network performance. The Netflix cloud services, for example, use BBR and found it can improve throughput threefold during heavy packet loss. Understanding how these algorithms react under different network conditions is an important activity when analyzing TCP performance.

Linux 5.6, released in 2020, added support for developing new congestion control algorithms in BPF [Corbet 20]. This allows them to be defined by the end user and loaded on demand.

### Initial Window
The initial window (IW) is the number of packets a TCP sender will transmit at the beginning of a connection before waiting for acknowledgment from the sender. For short flows, such as typical **HTTP** connections, an IW large enough to span the transmitted data can **greatly reduce completion time, improving performance**. Larger IWs, however, can risk congestion and packet drops. This is especially compounded when multiple flows start up at the same time.

The Linux default (10 packets, aka IW10) can be too high on slow links or when many connections start up; other operating systems default to 2 or 4 packets (IW2 or IW4).

## UDP
A major use for UDP has been DNS. Due to the simplicity of UDP, a lack of congestion control, and Internet support (it is not typically firewalled) there are now new protocols built upon UDP that implement their own congestion control and other features. An example is **QUIC**.

## QUIC and HTTP/3
QUIC is a network protocol designed by Jim Roskind at Google as a higher-performing, lower-latency alternative to TCP, optimized for HTTP and TLS 

QUIC is built upon UDP, and provides several features on top of it, including:
- The ability to multiplex several application-defined streams on top of the same “connection.”
- A TCP-like reliable in-order stream transport that can be optionally turned off for individual substreams.
- Connection resumption when a client changes its network address, based on cryptographic authentication of connection IDs.
- Full encryption of the payload data, including QUIC headers.
- 0-RTT connection handshakes including cryptography (for peers that have previously communicated).

QUIC is in heavy use by the Chrome web browser.

# Hardware (10.4.2)

=======================================================================================================
# Terminology
**File system**: An organization of data as files and directories, with a file-based interface for accessing them, and file permissions to control access. Additional content may include special file types for devices, sockets, and pipes, and metadata including file access timestamps.

**File system cache**: An area of main memory (usually DRAM) used to cache file system contents, which may include different caches for various data and metadata types.

**Operations**: File system operations are requests of the file system, including read(2), write(2), open(2), close(2), stat(2), mkdir(2), and other operations.

**I/O**: Input/output. File system I/O can be defined in several ways; here it is used to mean only operations that directly read and write (performing I/O), including read(2), write(2), stat(2) (read statistics), and mkdir(2) (write a new directory entry). I/O does not include open(2) and close(2) (although those calls update metadata and can cause indirect disk I/O).

**Logical I/O**: I/O issued by the application to the file system.

**Physical I/O**: I/O issued directly to disks by the file system (or via raw I/O).

**Block size**: Also known as record size, is the size of file system on-disk data groups. See Block vs. Extent in Section 8.4.4, File System Features.

**Throughput**: The current data transfer rate between applications and the file system, measured in bytes per second.

**inode**: An index node (inode) is a data structure containing metadata for a file system object, including permissions, timestamps, and data pointers.

**VFS**: Virtual file system, a kernel interface to abstract and support different file system types.

**Volume**: An instance of storage providing more flexibility than using a whole storage device. A volume may be a portion of a device, or multiple devices.

**Volume manager**: Software for managing physical storage devices in a flexible way, creating virtual volumes for use by the OS.

# Models
## File System Interfaces
![File System Interfaces](./images/Ch8/Ch8-File-System-Interfaces.png)

## File System Cache
![File System Cache](./images/Ch8/Ch8-File-System-Cache.png)

## Second Level Cache
![Second Level Cache](./images/Ch8/Ch8-File-System-Second-Level-Cache.png)

# Concepts
File system latency is the primary metric of file system performance, measured as the time from a logical file system request to its completion. It includes time spent in the file system and disk I/O subsystem, and waiting on disk devices—the physical I/O. Application threads often block during an application request in order to wait for file system requests to complete—in this way, file system latency directly and proportionally affects application performance.

Cases where applications may not be directly affected include the use of non-blocking I/O, prefetch (Section 8.3.4), and when I/O is issued from an asynchronous thread (e.g., a background flush thread)

# Architecture
## File System I/O Stack
![File System I/O Stack](./images/Ch8/Ch8-File-System-IO-Stack.png)

# Methodology
## Latency Analysis
| Layer | Pros | Cons |
| :---    | :---     |  :---     |
| **Application** | Closest measure of the effect of file system latency on the application; can also inspect application context to determine if latency is occurring during the application’s primary function, or if it is asynchronous. | Technique varies between applications and application software versions. |
| **Syscall interface** | Well-documented interface. Commonly observable via operating system tools and static tracing. | Syscalls catch all file system types, including non-storage file systems (statistics, sockets), which may be confusing unless filtered. Adding to the confusion, there may also be multiple syscalls for the same file system function. For example, for read, there may be read(2), pread64(2), preadv(2), preadv2(2), etc., all of which need to be measured. |
| **VFS** | Standard interface for all file systems; one call for file system operations (e.g., vfs_write()) | VFS traces all file system types, including non-storage file systems, which may be confusing unless filtered. |
| **Top of file system** | Target file system type traced only; some file system internal context for extended details. | File system-specific; tracing technique may vary between file system software versions (although the file system may have a VFS-like interface that maps to VFS, and as such doesn’t change often). |

## Workload Characterization
Basic attributes that characterize the file system workload:
- Operation rate and operation types
- File I/O throughput
- File I/O size
- Read/write ratio
- Synchronous write ratio
- Random versus sequential file offset access

### Advanced Workload Characterization Checklist
Additional details may be included to characterize the workload. These have been listed here as questions for consideration, which may also serve as a checklist when studying file system issues thoroughly:
- What is the file system cache hit ratio? Miss rate?
- What are the file system cache capacity and current usage?
- What other caches are present (directory, inode, buffer), and what are their statistics?
- Have any attempts been made to tune the file system in the past? Are any file system parameters set to values other than their defaults?
- Which applications or users are using the file system?
- What files and directories are being accessed? Created and deleted?
- Have any errors been encountered? Was this due to invalid requests, or issues from the file system?
- Why is file system I/O issued (user-level call path)?
- To what degree do applications directly (synchronously) request file system I/O?
- What is the distribution of I/O arrival times?

## Performance Monitoring
Key metrics for file system performance are:
- Operation rate
- Operation latency

## Static Performance Tuning
Static performance tuning focuses on issues of the configured environment. For file system performance, examine the following aspects of the static configuration:
- How many file systems are mounted and actively used?
- What is the file system record size?
- Are access timestamps enabled?
- What other file system options are enabled (compression, encryption...)?
- How has the file system cache been configured? Maximum size?
- How have other caches (directory, inode, buffer) been configured?
- Is a second-level cache present and in use?
- How many storage devices are present and in use?
- What is the storage device configuration? RAID?
- Which file system types are used?
- What is the version of the file system (or kernel)?
- Are there file system bugs/patches that should be considered?
- Are there resource controls in use for file system I/O?

# Observability Tools
| Tool | Description |
| :---    | :---     |
| *mount* | List file systems and their mount flags |
| *free* | Cache capacity statistics |
| *top* | Includes memory usage summary |
| *vmstat* | Virtual memory statistics |
| *sar* | Various statistics, including historic |
| *slabtop* | Kernel slab allocator statistics |
| *strace* | System call tracing |
| *fatrace* | Trace file system operations using fanotify |
| *latencytop* | Show system-wide latency sources |
| *opensnoop* | Trace files opened |
| *filetop* | Top files in use by IOPS and bytes |
| *cachestat* | Page cache statistics |
| *ext4dist (xfs, zfs, btrfs, nfs)* | Show ext4 operation latency distribution |
| *ext4slower (xfs, zfs, btrfs, nfs)* | Show slow ext4 operations |
| *bpftrace* | Custom file system tracing |

## Other Tools
| Tool | Description |
| :---    | :---     |
| *syscount* | Counts syscalls including file system-related |
| *statsnoop* | Trace calls to stat(2) varieties |
| *syncsnoop* | Trace sync(2) and variety calls with timestamps |
| *mmapfiles* | Count mmap(2) files |
| *scread* | Count read(2) files |
| *fmapfault* | Count file map faults |
| *filelife* | Trace short-lived files with their lifespan in seconds |
| *vfsstat* | Common VFS operation statistics |
| *vfscount* | Count all VFS operations |
| *vfssize* | Show VFS read/write sizes |
| *fsrwstat* | Show VFS reads/writes by file system type |
| *fileslower* | Show slow file reads/writes |
| *filetype* | Show VFS reads/writes by file type and process |
| *ioprofile* | Count stacks on I/O to show code paths |
| *writesync* | Show regular file writes by sync flag |
| *writeback* | Show write-back events and latencies |
| *dcstat* | Directory cache hit statistics |
| *dcsnoop* | Trace directory cache lookups |
| *mountsnoop* | Trace mount and umounts system-wide |
| *icstat* | Inode cache hit statistics |
| *bufgrow* | Buffer cache growth by process and bytes |
| *readahead* | Show read ahead hits and efficiency |

# References from book reading google group
1. [Linux Performance Tuning: Dealing with Memory and Disk IO](https://www.yugabyte.com/blog/linux-performance-tuning-memory-disk-io/)
2. 
