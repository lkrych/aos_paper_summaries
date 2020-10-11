# Limits to Low-Latency Communications on High-Speed Networks

### Authors: Chandramohan A. Thekkath and Henry M. Levy from the University of Washington

#### Published: ACM Transactions on Computer Systems, 11(2), 179-203 (1993)

## Table of Contents

* [Introduction](#introduction)
* [Network Technologies](#network-technologies)
* [RPC](#rpc)
* [Experimental Setup](#experimental-setupj)
* [Results](#results)
* [Conclusion](#conclusion)
* [Sources](#sources)

### Introduction

The orchestrated dance that takes one bit of information from the memory of one computer down into the depths of a network card, onto the physical media that transmits it across space, and up into the memory of another computer is a modern miracle. At the time this paper was written, new and exciting methodologies for organizing computer networks and designing distributed systems were coming into vogue. In this paper, the authors, Thekkath and Levy, experimentally review the efficiency of different hardware/network technologies by measuring a bespoke RPC implementation across the systems. 

### Network Technologies

The authors used three different local area network technologies to connect the machines they ran their experiments on.

* **Ethernet** -  The most common LAN technology. It is a frame-based broadcast protocol that uses collision detection protocols to avoid simultaneous transmission and loss of messages. The particular Ethernet standard used by the authors was a 10Mbit/sec LAN.

* **Token-Ring** - Another frame-based LAN technology that uses a special frame to denote whose turn it is to transmit a message. This ensures that each participant in the network has a turn to talk and that no one tries to talk over each other. The token-ring system used by the authors was a 100 Mbit/sec fiber-optic implementation called Fiber Distributed Data Interface (FDDI).

* **ATM (Asynchronous Transfer Mode)** - is a cell-based (fixed-length) networking technology that can be used in both LANs and WANs. ATM uses virtual circuits to transfer cells from one host to the next. The ATM implementation used by the authors had cell sizes of 53 bytes, and ran at 140 Mbit/sec.

### RPC

To ensure that they were really testing the limitations of the networking technologies/hardware setups, and not the software overhead of marshalling, stubs and packet exchange protocols, the authors crafted an efficient RPC system. The implementation was written entirely in C. 

One of the techniques used by the authors to speed up marshaling was to have the marshaling done in the kernel instead of user space. The stubs linked in the user address space just trapped into the kernel. Another improvement is that during control transfer, the authors RPC system defers blocking the client thread on a send. Instead, the client thread spin-waits for a short period before being moved to the sleep queue. If the server response time is small, the reply is received immediately by the spinning client.


### Experimental Setup

The testbed for this paper tried to isolate the performance of the controller and the network link, it simply sent and received packets on the network without OS intervention. The testbed hardware consisted of two workstations, either two DECstations or two Sparcstations, connected through an isolated network. The Sparcstations had faster processors. The DECstations were connected to ethernet, FDDI ring, and an ATM network. The Sparstations were connected with Ethernet.

### Results

Two sets of experiments were run in this paper. In the first set of experiments, the authors measured the performance of each configuration in sending packets from one node to the other. In the second set of experiments that authors measured the performance of their RPC implementation on the same hardware/network setups.

Let's discuss the first set of experiments. In these experiments, the authors attempted to measure the hardware level round-trip-time (RTT) of packets across the experimental setups. As mentioned earlier, there were four systems: Ethernet, FDDI, and ATM on the DECstations, and Ethernet on the Sparcstations. The packets were sent and received from host memory, this means that the cost of moving data over the host bus is including in measurement. 

The first set of results correspond to the relationship between throughput and latency. For small packets, the ethernet systems were faster in RTT than the much higher bandwidth FDDI, despite the 10-fold bandwidth advantage of FDDI. The ATM network was 3x faster than both of these for a single cell transfer, however, this measurement ignores switching delays, which would admittedly be much larger in a non-trivial setup. For large packets, the high bandwidth capacity of ATM and FDDI improved their latency times markedly compared to the Ethernet setups. 

The second set of results correspond to the effect the controller has on latency in the packet exchange. These controllers are microchips that help manage communication between the CPU and peripheral devices. In the Sparcstation setup, the controller uses direct memory access (DMA) transfers, which means it is able to overlap the data transfer over the bus with the transfer onto the network. The DECstations by contrast need to copy the data over the bus before the controller can begin the transfer. This means that there isn't much controller latency in the Sparcstation exchange of larger packets, while the DECstations need to contend with copying more data.

The second set of results in this paper correspond to the measurements of RPC across these systems. The goal in this section of the experiments is to show that RPC can be structured so that effective hardware costs are significant when it comes to latency. 

Overall, the absolute increase contributed by RPC is comparable in both Ethernet and ATM, where it adds about 130% to the cost of low-level messages. In FDDI ring, the higher-level RPC functions cost more because the controller/host interface complicates the memory management and the code path for the controller is more convoluted than in simple packet exchange. 

### Conclusion

When considering the efficiency of distributed systems, many different factor must be kept in mind. The authors chose to focus on three issues, the cost of memory copying, the cost of interrupt handling, and the costs of network design.

The main difference in memory copying comes in the scheme used by the network controllers, do they use programmed I/O (PIO) or DMA? For short packets the differences are negligible, but for larger packets there is a point where programmed I/O will be slower than DMA.

One problem with DMA however is that it might extract a heavier penalty than PIO due to cache effects. If the cache does not snoop on the operations, cache blocks could be left incoherent as a result of the DMA operations. This means that cache needs to be flushed, which will slow down the processes trying to execute. Newer processors provide memory coherence for DMA. 

Lastly, when it comes to network design, even though FDDI token ring has much higher bandwidth, it has latency increases that grow with the number of stations in the network. Comparing this setup to a growing Ethernet LAN, and the Ethernet latency remains practically constant. ATM-style networks function much better, but this might just be because the experimenters used a relatively naive ATM setup that ignored the practical costs of the fragmenting and routing that happens at different nodes in the network.

### Sources

1. Thekkath, C. A., &amp; Levy, H. M. (1993). Limits to low-latency communication on high-speed networks. ACM Transactions on Computer Systems, 11(2), 179-203. doi:10.1145/151244.151247