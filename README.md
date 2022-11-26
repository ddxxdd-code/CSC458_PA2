# CSC458_PA2
## Instructions to run
To reproduce the results, run `sudo ./run.sh`. 

It takes about 5 minutes to run one experiment with one buffer size, so roughly 10 minutes to produce all results. The results are included as log file and plots in folders `bb-q20` and `bb-q100`.

## Questions and answers
### 1. Why do you see a difference in webpage fetch times with small and large router buffers?

- Buffer size 20: fetch average = 0.500966666667s fetch std dev = 0.0755140825718

- Buffer size 100: fetch average = 1.26273015873s fetch std dev = 0.517863188191

The reason is bufferbloat happened here. When the router buffer is small, it fills up quickly, so TCP can detect the drop then adjust to appropriate sending rate for the network which minimizes packet lost and queuing delay. While for the case of large buffer, the packets sent just go to the buffer and the sender is unaware of the bottleneck link. In this case, the sender just send at high rate, which yields high rate of loss and retransmission. Also, with the TCP congestion control, it backs off more often, taking more time in recovery. To make it even worse, the larger buffer gives more queuing delay for the packets in the buffer and this also adds to the total delay. Therefore, we see much higher fetch times with larger buffer size.

### 2. Bufferbloat can occur in other places such as your network interface card (NIC). Check the output of ifconfig eth0 on your VirtualBox VM. What is the (maximum) transmit queue length on the network interface reported by ifconfig? For this queue size and a draining rate of 100 Mbps, what is the maximum time a packet might wait in the queue before it leaves the NIC?

Running `ifconfig eth0` on the VM gives
> eth0      Link encap:Ethernet  HWaddr 08:00:27:ea:8e:68  
> inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
> UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
> RX packets:79773 errors:0 dropped:0 overruns:0 frame:0
> TX packets:30286 errors:0 dropped:0 overruns:0 carrier:0
> collisions:0 txqueuelen:1000 
> RX bytes:72448586 (72.4 MB)  TX bytes:5361474 (5.3 MB)

Therefore, the transmit queue length, `txqueuelen`, is 1000. The maximum time a packet moght waite is the time it takes to empty all the 1000 packets. The size of a packet is specified by MTU, which is `1500` in bytes. Hence the time needed is `queue_length * packet_size / draining_rate = 1000 * 1500 bytes/ (100*10^6)bps = 12Mb / 100Mbps = 0.12s`.

### 3. How does the RTT reported by ping vary with the queue size? Write a symbolic equation to describe the relation between the two (ignore computation overheads in ping that might affect the final result).

RTT = transmision delay + propogation delay + queuing delay.
From the graph of RTT at maximum queue size 20 and maximum queue size 100, RTT at maximum queue size 20 is around 150ms, RTT at maximum queue size 100 is around 200ms, queuing delay is positively relates to queue size.

So we have `RTT = C + k * queue size`. Where C describes the delay of propagation, transmission, and processing a packet, which is purely dependent on the network, links, and switches. While the part of `k * queue size` is dependent of the queue size.

### 4. Identify and describe two ways to mitigate the bufferbloat problem.

The bufferbloat problem happens when there is a small bandwidth link with large buffer and sender not aware of the buffer is filled. 

- One possibility is to reduce the maximum buffer size allowed in a network so no really large buffer presents in the network. The total size of buffer will increase the queuing time for a packet and cause bufferbloat. So no extremely large buffer can reduce the risk of having the prolem of bufferbloat.
- Another possibility is to modify the congestion control strategy. To introduce queue eviction strategy before the buffer is full to warn possible buffer overflow or to send message to sender when buffer space is shrinking to backoff earlier. By reporting to sender earlier than buffer filled up can reduce the sending rate and avoid bufferbloat.
