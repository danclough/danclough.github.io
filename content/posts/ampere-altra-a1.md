+++
date = 2021-12-31
title = "Ampere Altra vs Raspberry Pi 4"
slug = "ampere-altra-a1"
cover = "images/2021/12/jainath-ponnala-BIHgNEaM394-unsplash.jpg"
tags = ["arm","cloud","raspberrypi"]
+++

In early 2020, recent semiconductor startup Ampere announced the Altra, an ultra-dense 80-core ARM64 CPU targeted at cloud computing environments.  Patrick Kennedy of ServeTheHome covered the release with an [excellent in-depth article](https://www.servethehome.com/ampere-altra-80-arm-cores-for-cloud/) last year which I highly recommend reading.

In mid-2020, Oracle became the first cloud provider to add the Ampere Altra to their cloud computing lineup.  And in early 2021, Oracle took the unusual step of adding the Altra A1 VMs to their "Always Free" tier, allowing anyone to create ARM64 VMs with up to 4 cores and 24GB of RAM at no cost.  I've recently started playing around more with my Oracle Cloud account, and decided to use Terraform to spin up a free ARM64 VM to compare its performance with one of my existing Raspberry Pi 4Bs.
<!--more-->
{{< figure src="/images/2021/12/Ampere-Altra-Processor-Complex.jpg" position="center" caption="Overview of Ampere Altra specs, courtesy of Patrick Kennedy at <a href=\"https://www.servethehome.com\">ServeTheHome</a>." >}}

## Specifications
Oracle allows "Always Free" tier users to create flexible instance sizes from a total resource pool of 4 "OCPUs" and an astonishing **24GB** of RAM.  To make the test as fair as possible, I created an instance with 4 OCPUs and 8GB of RAM to match the 8GB RPi 4 that I tested locally.

|**System**|**VM.Standard.A1.Flex**|**Raspberry Pi 4B**|
|:-|:-|:-|
|CPU|4x Ampere Altra A1 @ 3.0GHz|4x ARM Cortex-A72 @ 1.5GHz|
|RAM|8GB|8GB|
|Storage|50GB boot volume|32GB microSD|
|OS|Ubuntu 20.04 ARM64|Ubuntu 20.04 ARM64|

## Benchmark #1 - UnixBench
UnixBench is, as the name implies, a benchmark for Unix systems using standard Unix operations.  It's intended to test multiple aspects of the system as a whole, both hardware and software, rather than focusing on any one particular component.

The standard testing methodology and aggregation of multiple aspects of system performance in a single score make it a very reliable method of judging system performance across platforms and CPU architectures.

#### Ampere Altra
```
   BYTE UNIX Benchmarks (Version 5.1.3)
        
   System: **********: GNU/Linux
   OS: GNU/Linux -- 5.11.0-1022-oracle -- #23~20.04.1-Ubuntu SMP Fri Nov 12 15:45:47 UTC 2021
   Machine: aarch64 (aarch64)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   18:42:15 up 2 min,  1 user,  load average: 0.28, 0.27, 0.11; runlevel 2021-12-30
        
------------------------------------------------------------------------
Benchmark Run: Thu Dec 30 2021 18:42:15 - 19:10:10
4 CPUs in system; running 1 parallel copy of tests
        
Dhrystone 2 using register variables       41483611.9 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     8466.2 MWIPS (9.9 s, 7 samples)
Execl Throughput                               5163.9 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks       1077137.2 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks          300831.1 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks       3080256.7 KBps  (30.0 s, 2 samples)
Pipe Throughput                             1960270.1 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                  87688.7 lps   (10.0 s, 7 samples)
Process Creation                               7644.5 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                  10008.2 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                   3001.8 lpm   (60.0 s, 2 samples)
System Call Overhead                        1649115.6 lps   (10.0 s, 7 samples)
        
System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   41483611.9   3554.7
Double-Precision Whetstone                       55.0       8466.2   1539.3
Execl Throughput                                 43.0       5163.9   1200.9
File Copy 1024 bufsize 2000 maxblocks          3960.0    1077137.2   2720.0
File Copy 256 bufsize 500 maxblocks            1655.0     300831.1   1817.7
File Copy 4096 bufsize 8000 maxblocks          5800.0    3080256.7   5310.8
Pipe Throughput                               12440.0    1960270.1   1575.8
Pipe-based Context Switching                   4000.0      87688.7    219.2
Process Creation                                126.0       7644.5    606.7
Shell Scripts (1 concurrent)                     42.4      10008.2   2360.4
Shell Scripts (8 concurrent)                      6.0       3001.8   5003.1
System Call Overhead                          15000.0    1649115.6   1099.4
                                                                   ========
System Benchmarks Index Score                                        1669.7
        
------------------------------------------------------------------------
Benchmark Run: Thu Dec 30 2021 19:10:10 - 19:38:05
4 CPUs in system; running 4 parallel copies of tests
        
Dhrystone 2 using register variables      165260124.9 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                    33900.5 MWIPS (9.9 s, 7 samples)
Execl Throughput                              12794.4 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        776156.4 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks          214667.2 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks       2345518.7 KBps  (30.0 s, 2 samples)
Pipe Throughput                             7825529.6 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 867609.5 lps   (10.0 s, 7 samples)
Process Creation                              20049.6 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                  24471.2 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                   3410.3 lpm   (60.0 s, 2 samples)
System Call Overhead                        4281214.8 lps   (10.0 s, 7 samples)
        
System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0  165260124.9  14161.1
Double-Precision Whetstone                       55.0      33900.5   6163.7
Execl Throughput                                 43.0      12794.4   2975.5
File Copy 1024 bufsize 2000 maxblocks          3960.0     776156.4   1960.0
File Copy 256 bufsize 500 maxblocks            1655.0     214667.2   1297.1
File Copy 4096 bufsize 8000 maxblocks          5800.0    2345518.7   4044.0
Pipe Throughput                               12440.0    7825529.6   6290.6
Pipe-based Context Switching                   4000.0     867609.5   2169.0
Process Creation                                126.0      20049.6   1591.2
Shell Scripts (1 concurrent)                     42.4      24471.2   5771.5
Shell Scripts (8 concurrent)                      6.0       3410.3   5683.8
System Call Overhead                          15000.0    4281214.8   2854.1
                                                                   ========
System Benchmarks Index Score                                        3641.0
```

#### Raspberry Pi 4
```
   BYTE UNIX Benchmarks (Version 5.1.3)
        
   System: **********: GNU/Linux
   OS: GNU/Linux -- 5.4.0-1047-raspi -- #52-Ubuntu SMP PREEMPT Wed Nov 24 08:16:38 UTC 2021
   Machine: aarch64 (aarch64)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   11:40:13 up 3 days,  1:39,  1 user,  load average: 1.57, 0.89, 0.74; runlevel 2021-09-07
        
        ------------------------------------------------------------------------
Benchmark Run: Thu Dec 30 2021 15:42:07 - 16:10:28
4 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables       15090554.5 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     2682.9 MWIPS (9.9 s, 7 samples)
Execl Throughput                                570.5 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks         92917.6 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           25770.6 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        273542.6 KBps  (30.0 s, 2 samples)
Pipe Throughput                              141626.1 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                  29427.5 lps   (10.0 s, 7 samples)
Process Creation                               2160.1 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   1980.0 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    670.6 lpm   (60.0 s, 2 samples)
System Call Overhead                         170070.1 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   15090554.5   1293.1
Double-Precision Whetstone                       55.0       2682.9    487.8
Execl Throughput                                 43.0        570.5    132.7
File Copy 1024 bufsize 2000 maxblocks          3960.0      92917.6    234.6
File Copy 256 bufsize 500 maxblocks            1655.0      25770.6    155.7
File Copy 4096 bufsize 8000 maxblocks          5800.0     273542.6    471.6
Pipe Throughput                               12440.0     141626.1    113.8
Pipe-based Context Switching                   4000.0      29427.5     73.6
Process Creation                                126.0       2160.1    171.4
Shell Scripts (1 concurrent)                     42.4       1980.0    467.0
Shell Scripts (8 concurrent)                      6.0        670.6   1117.6
System Call Overhead                          15000.0     170070.1    113.4
                                                                   ========
System Benchmarks Index Score                                         265.5

------------------------------------------------------------------------
Benchmark Run: Thu Dec 30 2021 16:10:28 - 16:38:54
4 CPUs in system; running 4 parallel copies of tests

Dhrystone 2 using register variables       60067639.9 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                    10702.3 MWIPS (9.8 s, 7 samples)
Execl Throughput                               1995.7 lps   (29.9 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        181967.0 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           48675.5 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        455783.0 KBps  (30.0 s, 2 samples)
Pipe Throughput                              559095.7 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 127558.3 lps   (10.0 s, 7 samples)
Process Creation                               5773.4 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   5510.9 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    746.7 lpm   (60.2 s, 2 samples)
System Call Overhead                         662741.3 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   60067639.9   5147.2
Double-Precision Whetstone                       55.0      10702.3   1945.9
Execl Throughput                                 43.0       1995.7    464.1
File Copy 1024 bufsize 2000 maxblocks          3960.0     181967.0    459.5
File Copy 256 bufsize 500 maxblocks            1655.0      48675.5    294.1
File Copy 4096 bufsize 8000 maxblocks          5800.0     455783.0    785.8
Pipe Throughput                               12440.0     559095.7    449.4
Pipe-based Context Switching                   4000.0     127558.3    318.9
Process Creation                                126.0       5773.4    458.2
Shell Scripts (1 concurrent)                     42.4       5510.9   1299.7
Shell Scripts (8 concurrent)                      6.0        746.7   1244.5
System Call Overhead                          15000.0     662741.3    441.8
                                                                   ========
System Benchmarks Index Score                                         730.7
```

## Benchmark #2 - Sysbench

Sysbench is an open-source benchmarking tool that features several independent tests of system hardware performance.  The tests I'll focus on are CPU, memory, and file I/O.

Each test was run on a clean system with no other competing workloads.

|**Category**|**Metric**|**Ampere Altra A1**|**Raspberry Pi 4B**|
|:-|:-|:-|:-|
|CPU|primes/sec|3511.65|1486.26|
|RAM|throughput, MiB/s|4643.80|2016.59|
|File I/O|sequential write, MiB/s|55.81|13.02|
|File I/O|sequential write, iops|3551.19|797.32|
|File I/O|random write, MiB/s|33.17|4.59|
|File I/O|random write, iops|2122.76|293.73|

# Interpreting the Results
Straight away, it's clear that there is a significant performance difference between the admittedly aging ARM Cortex-A72 and the Altra.

On CPU performance, adjusting for the clock speed difference between the Altra and the Cortex-A72 yields an 18% improvement in IPC.  Considering the 4 year age difference between the two chips, it's not earth-shattering - x86 platforms have shown similar trends over the years - but it's still respectable and shows that Ampere has the engineering talent to keep pace with the likes of Intel and AMD.

Since UnixBench is intended to be a general system benchmark and not purely a measure of CPU muscle, I was most interested to see what else contributed to the whopping 5x increase in the UnixBench index score.  For that, I believe the Sysbench memory test points us in the right direction.

The Sysbench RAM result shows an incredible **130% increase** in memory throughput on the A1.  This result is surprising on its surface - after all, both the Altra and the Cortex-A72 use DDR4 RAM, right?  Not quite.

As a small, low-power/embedded style device, the Raspberry Pi 4 carries the burden of an insanely low power budget - less than 10W.  Cutting consumption on the CPU alone isn't enough to meet those demands, however, so the Pi 4 uses a low-power variant of DDR4 called **LPDDR4**, intended primarily for mobile platforms.  

As discussed in [this DDR4 explainer](https://www.hardwaretimes.com/lpddr4-vs-ddr4-vs-lpddr4x-memory-whats-the-difference/) from HardwareTimes, LPDDR4 makes several key trade-offs for power consumption that result in greatly decreased memory bandwidth.  While the Pi 4's DRAM operates at the same 3200MT/s rate as the Altra, its performance is ultimately kneecapped by a single-channel memory implementation and LPDDR4's greatly reduced 32-bit bus width.

{{< figure src="/images/2021/12/50120709591_4c50126bda_c.jpg" position="center" caption="The lone LPDDR4 package on the Raspberry Pi 4B.<br/>Photo by <a href=\"https://flic.kr/p/2jmZJMn\">Jeff Geerling</a>, licensed under <a href=\"https://creativecommons.org/licenses/by/2.0/\">CC BY 2.0</a>" >}}

Other factors like File I/O do show significant improvements from the Pi 4, but that seems to be due solely to the Pi 4's microSD card rather than some exceptional storage technology within the Oracle Cloud.  Keep in mind that 55MiB/s and 2-3k IOPS is lacking compared to even the lowest consumer-grade SATA 6Gb/s SSDs.  It is noteworthy nonetheless that Oracle is able to provide this level of performance in a free cloud server.  I fully expect that the Pi 4 would win in this category with a USB3 flash drive or SSD.

All that being said, I went into this experiment fully expecting to see performance of the free Ampere VM roughly on-par with the Pi 4, at best.  My suspicions were based on the anemic performance of other cloud providers' free tier compute instances - AWS's t2.micro, Azure's B1s burstable, and even Oracle's existing AMD Epyc "micro" shape.  The Ampere VMs in Oracle's free tier are surprisingly capable and unlike any other free offering I've seen to date.



