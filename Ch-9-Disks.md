Disk I/O can cause significant application latency, and is therefore an important target of systems performance analysis. Under high load, disks become a bottleneck, leaving CPUs idle as the system waits for disk I/O to complete. Identifying and eliminating these bottlenecks can improve performance and application throughput by orders of magnitude.

The term disks refers to the primary storage devices of the system. They include flash-memory-based solid-state disks (SSDs) and magnetic rotating disks. SSDs were introduced primarily to improve disk I/O performance, which they do. However, demands for capacity, I/O rates, and throughput are also increasing, and flash memory devices are not immune to performance issues

# Terminology
**Virtual disk**: An emulation of a storage device. It appears to the system as a single physical disk, but it may be constructed from multiple disks or a fraction of a disk.

**Transport**: The physical bus used for communication, including data transfers (I/O) and other disk commands.

**Sector**: A block of storage on disk, traditionally 512 bytes in size, but today often 4 Kbytes.

**I/O**: Strictly speaking, I/O includes only disk reads and writes, and would not include other disk commands. I/O can be described by, at least, the direction (read or write), a disk address (location), and a size (bytes).

**Disk commands**: Disks may be commanded to perform other non-data-transfer commands (e.g., a cache flush).

**Throughput**: With disks, throughput commonly refers to the current data transfer rate, measured in bytes per second.

**Bandwidth**: This is the maximum possible data transfer rate for storage transports or controllers; it is limited by hardware.

**I/O latency**: Time for an I/O operation from start to end. Within below sections, Measuring Time, defines more precise time terminology. Be aware that networking uses the term latency to refer to the time needed to initiate an I/O, followed by data transfer time.

**Latency outliers**: Disk I/O with unusually high latency.

# Models


# References from book reading google group
1. [Linux Performance Tuning: Dealing with Memory and Disk IO](https://www.yugabyte.com/blog/linux-performance-tuning-memory-disk-io/)
2. 
