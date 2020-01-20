# chapter 12

## The rfcs

Base:
  793  -- The original specification for TCP
          (updated by 1122, 3168, 6093, 6528)
     1122  -- Host Requirements RFC
     3168  -- The Addition of Explicit Congestion Notification (ECN) to IP
     6093  -- On the Implementation of the TCP Urgent Mechanism
     6528  -- Defending against Sequence Number Attacks

Extensions
    7323 -- TCP Extensions for High Performance
            (WS and Timestamp)
    2018 -- SACK
    5482 -- TCP User Timeout Option
    5925 -- TCP Auth option

Congestion Control
 5681  -- TCP Congestion Control
 3782  -- New Reno modification to TCP's fast recovery
 6675 (3517) -- Conservative Loss Recovery Algorithm based on SACK
 3390  -- Increasing TCP's Initial Window
 3168  -- The Addition of Explicit Congestion Notification (ECN) to IP

Retrasnmission Timeouts
 6298  -- Computing TCP's Retransmission Timer
 5682  -- F-RTO recovery
 4015  -- The Eifel Response Algorithm for TCP

NAT
 5382  -- NAT Behavioral Requirements for TCP

Ack behavior
 2883  -- An Extension to the Selective Acknowledgement (SACK) Option for TCP

Security:
  6056 -- Recommendations for Transport-Protocol Port Randomization
  5926 -- Cryptographic Algorithms for the TCP Authentication Option (TCP-AO)
  5927 -- ICMP Attacks against TCP

Conn Mgmt

Urg mechanism guidelines
  6093 -- On the Implementation of the TCP Urgent Mechanism

Retrans Behavior:
  5827 -- Early Retransmit for TCP & SCTP
  3708 -- Using dup-SACKs to detect spurious retrans

Congestion Detection & Control
  5690 --  Adding Acknowledgement Congestion Control to TCP
  5562 --  Adding (ECN) Capability to TCP's SYN/ACK Packets
  4782 --  Quick-Start for TCP and IP
  3649 --  HighSpeed TCP for Large Congestion Windows
  2861 --  TCP Congestion Window Validation

MTCP
  6182 -- Architectural Guidelines for Multipath TCP Development

PMTU
  1191
  1981

# chapter 13

## Connection establishment

* 4 tuples
* 3 phases - setup, data-transfer and teardown
* active opener and passive opener
    * TCP allows for simultaneous open, but its very very rare.
* tcp supports carrying data on syn-segment, but its
  rarely used as the sockets api didn't support it.
* active close can be initiated by either party.
* 3 segments to establish and 4 to close.
* half-close is not that popular, but possible using sockets api
    * call shutdown() instead of close()

## Sequence numbers

* Seq number tracks every byte of data. SYN and FIN count as 1 seq number too.
* Sender sequence is the sequence number of the first byte.
* Ack sequence is the number that it expects to receive next

## ISN consideration

* Change every 4us by 1.
* Diff ISN ensure the next incarnation of one 4-tuple doesn't interfere with previous
* linux does it in a semi-random way
    * clock + hash of the 4 tuple + a number to the hash that changes every 5 mins
    * top 8 bits are sequence number of secret, remaining bits are the hash
    * effectively, the ISN is difficult to guess, and it increases over time

## Conn Establishment(SYN) timeout

* To simulate this, you should fake a arp for the host (otherwise the IP layer itself wont make it out)
* There is an exponential backoff
    * 3, 6, 12, 24, 48s timeout.
    * On top of this, there is a small randomization so that connections at the same time, dont burst it
      out all in sync.
* no of times to retry is a configurable - net.ipv4.tcp_syn_retries

## TCP options

* only 40 more bytes possible. So not much leeway.

| Option         | How big        | Description and purpose           |
| -------------  | -------------- | --------------------------------- |
| EOL            | 1              | Signifies end of option           |
| NOP            | 1              | Used for padding                  |
| MSS            | 4              | maximum segment size              |
| WSOPT          | 3              | window scaling factor             |
| SACK-Permitted | 2              | sender supports sack              |
| SACK           | Var            | sack block                        |
| TSOPT          | 10(!)          | timestamp                         |
| UTO            | 4              | user-timeout                      |
| TCP-AO         | Var            | TCP-auth option .. (not popular?) |

### MSS

* each size sends it mss. Its a hard-limit. No negotiation.
* but path mss can be smaller. DF bit is usually set in tcp to know the path-MTU anyway.
* Usually present only in SYN. But can be present in other packets too (to confirm)

### SACK

* SACK-Permitted announces the ability in SYN. If both sides advertise, this is on.
* SACK is 8n+2 long. type/len/and 2 sequence nos.
    * Typically on 3 blocks are possible if TSOPT is also present.

### WSOPT

* the value has a number from 1 to 14, that left-shifts the recv-window by that number.
    * when 0(no scale), max = 65535 (2**16)
    * when 14,          max = 1G
* note, the scale factor is fixed at SYN/SYN-ACK. Can be diff in each direction
* The receiver of SYN-ACK should see the option, otherwise it resets to 0.
* linux sets this automatically based on sock-buff size

### TSOPT

* TSVal - sender timestamp, TSER/TSecr - Time Stamp Echo Reply
* Recipient just echoes back the TSVal in its Ack in TSecr
    * Thus the actual RTT measured in 2 leg-times + time it took for the
      responder to send its ack.
    * But this is convenient as that shows the total amount of latency anyway.
* PAWS - protection against wrapped sequence numbers.
    * Timestamp is a 32bit extenstion of the seq number.
    * If a seg arrives again with same value it can be discarded as the
      timestamp will still be that of the old packet.
    * Although the receiver doesn't interpret TS-value, it expects it to increase.

### UTO

* Amount of time a sender is willing to wait for an ACK before abandoning the
  connection
* It can help in remote ends to hold on to a connection longer and for intermediate
  NAT devices to set their activity timers.
* It is just advisory.
* The option value is expressed as a 15-bit value in units of seconds or minutes
  following a bit field (“granularity”) that indicates that the value is in
  minutes (1) or seconds (0)
* Shorter values = connection torn down easily (DOS attacks).
  Larger values = resource exhaustion. So each end has a upper/lower bound
* Sent in SYN or in any pkt when the values is adjusted locally.

### TCP-AO

* Tcp auth option. It requires each party to know a shared key apriori.

## PMTU

* SMSS - sender MSS
* PMTUD - Path MTU Detection
    * Fancy term for detecting PMTU using DF-bit-Fragmentation-Reqd/Packet-Too-Big messages
* PLPMTUD - Packetization Layer Path MTU Discovery
    * Detecting PMTU using pkt-losses instead of ICMP like messages.
* Initially we try with min(Interface-MTU, Sender-sent MSS or stored MSS for a host)
* Based on ICMP messages, we reduce. The entry may tell the next value to try itself or we can guess.
* TCP will send all pkts with DF bit.
* MTU might increase. So every timeout(~10 minutes), we can try higher value

## TCP State Transtions

```
c: Typical client transitions
s: Typical server transitions

a: active close
p: passive close

(S) -- very rare: Simultaneous open.

CLOSED
   |
   | c:open(a)
   |   send: SYN         c: recv: SYN/ACK
   |                          send: ACK
   +---- SYN_SENT  ---+-----------------------+
   |                  |                       |
   | s:open(p)        +--------+(S)           +--- ESTABLISHED
   |                           v              |
   +---- LISTEN ------------ SYN_RCVD --------+
               s: recv: SYN             s: recv: ACK
                   send: SYN/ACK


ESTABLISHED
   |
   | p: recv: FIN             p: close()                    p: recv:ACK
   |     send:ACK                  send:FIN
   +------------ CLOSE_WAIT ------------------- LAST_ACK ----------- (x)CLOSED
   |
   |
   |  a: close()
   |      send: FIN
   +--------- FIN_WAIT1
                  |
                  |  a: recv: ACK              a: recv: FIN
                  |                                 send: ACK
                  +--------------- FIN_WAIT2 ---------+
                  |                                   |
                  |  a: recv: FIN/ACK                 |
                  |       send: ACK                   |
                  +-----------------------------------+-- TIME_WAIT --- (x) CLOSED
                  |                                   |     (2 MSL)
                  |  a: recv: FIN                     |
                  |       send: ACK                   |
                  +----------------- CLOSING ---------+
                                                a: recv ACK

```

* MSL is the maximum time a segment can remain in the network.
    * It typically is set to 2 mins
    * Linux: net.ipv4.tcp_fin_timeout
    * Ideally one the same 4 tuple shouldn't be reused in this time.
        * Unfortunately some systems impose a stringent non-re-use of the local port
          if the local_port is part of a 4 tuple in TIME_WAIT state.

Read page number 620 onwards later to note this better

### Quiet Time

While 2MSL TIME_WAIT helps to avoid the 4 tuple re-use, this can be enforced only if the same OS is active. If the OS reboots, then it doesn't know how many 4 tuples were earlier in TIMED_WAIT.

OS'es wait for quiet time before starting TCP connections.
But usually reboot times are higher than 2MSL's anyway.

### Half Close

* The active endpoint is in the FIN_WAIT_2 state. The other end that still wishes
  to send data is in the `CLOSE_WAIT` state.
* There can be a timeout to wait for the other FIN - net.ipv4.tcp_fin_timeout

### Simultaneous Close

* Both FINs cross each other. Both sides are in FIN_WAIT_1 and then to CLOSING
* Both sides stay in TIME_WAIT state.

## Reset segments

### Conn Request to non-existant port

* UDP uses ICMP port unreachable. TCP uses RST
* RST for this case will have no seq number, but ACK will have SYN's ISN+1.

### Abort connection

* Orderly release and abortive release.
* Endpoint that sends RST, doeesn't wait for a reply
* Endpoint that receives RST locally closes, and notifies app (conn reset by peer)

## TIME_WAIT assasination

* A RST received for a 4 tuple that is in TIME_WAIT
* RST is simply ignored in this STATE

## Incoming Connection Queue

* OS has 2 queues - Connections in the SYN_RCVD state and
  connections in ESTABLISHED state, but not yet accepted by the application
* net.ipv4.tcp_max_syn_backlog -- SYN_RCVD connections  (system-wide across all
  listen sockets)
* backlog - per listen socket's length of ESTABLISHED conns
    * when backlog is full, OS doesn't respond to SYN.

## Attacks

*  SYN flood
    * Simply send SYN
    * We can send SYN-Cookies, so that we can rebuild info at First-ACK.
    * Server's ISN is a function of:
        * top 5 bits are (t mod 32) where t is a 32-bit counter that
          increases by 1 every 64s,
        * the next 3 bits are an encoding of the server’s MSS
          (one of eight possibilities), and
        * the remaining 24 bits are a server-selected cryptographic hash
          of the connection 4-tuple and t value.
    * One pitfall is the short duration for roll-over the connection.
      So if connection establishment takes `> 64s`, it is an issue.
* degradation attack on PMTUD
    * Attacker sends a very less PTB(Packet too Big) message
    * One soln is to not go less than 576
* Hijacking existing connection

# Chapter 14

* Retransmission happens either when
    * Retransmission Timeout happens
    * Fast Retransmit
        * When ACKs dont advance
        * When SACKs are received
* R1 and R2
    * defined in host-requirements rfc
    * R1 - number of retries to send a segment before a -ve advice is sent to IP layer.
    * R2 - timeout to abort the connection.
    * net.ipv4.tcp_retries1/2

## Setting RTO

### Classic method

* That of RFC 793 itself.
    ```
    SRTT ← α(SRTT) + (1 − α) RTT
    ```
    * α of 80-90% makes the above a low-pass filter or
      exponentially weighted moving average (EWMA).
* RTO is set to
    ```
    RTO = min(ubound, max(lbound,(SRTT)β))
    ```
    * β is the variance factor - usually 1.3 or 2.
    * lower bound is 1s, upper bound is 1min
* Peforms badly when there are wild fluctuations

### Standard Method

* Tracks both the mean of the RTT and the variance.
* The forumlae:
    ```
    M is the individual measurement.
    g is the α equivalent of classic method

    srtt ← (1 - g)(srtt) + (g)M
    rttvar ← (1 - h)(rttvar) + (h)(|M - srtt|)
    RTO = srtt + 4(rttvar)
    ```
* rttvar is the mean of the deviation.
* This is used to get the RTO instead of a fixed variance factor

### Retransmission Ambiguity and Karn’s Algorithm

* When retransmissions happen, we dont know if a ACK is that of first or retrans
    * Such ACKs that ack retransmitted data is not used for RTT calc.
* TCP applies a backoff factor to the RTO, which doubles each time a subsequent
  retransmission timer expires.
* Doubling continues until an acknowledgment is received for a segment that was
  not retransmitted. At this time, RTO is set to its old value.

### TSOPT

* TCP does not always provide an ACK for each segment it receives.
* In addition, when data is lost, reordered, or successfully retransmitted,
  the cumulative ACK mechanism of TCP means that there is not necessarily
  any fixed correspondence between a segment and its ACK.
* To handle these challenges, TCPs that use this option employ the following
  algorithm for taking RTT samples:

1. TSVal of a segment contains the current clock.
2. TSRecent = TSVal to send in the next ACK
   LASTAck  = Next seq number to expect/last ack sent.
3. When a new segment arrives,
    if its Seq == LastACK, then update TSRecent (perfect valid segment to arrive)
4. Whenever the receiver sends an ACK, put in TSRecent as TSER.
5. A sender receiving an ACK that advances its window:
    RTT = `current_clock` - TSEcr
    * Note this discards getting RTT from window-update ACKs
    * Say, we had a retransmission.
        * if the ack was generated on first segment itself, then we are good
          as this is indeed the RTT.
        * if the first was lost and the retrans was what triggered the ack, then
          also we are good, as this is indeed the RTT.
        * Because in both cases, the TSecr is from the pkt that triggers the ack.

### The Linux Method

Too intensive.. Needs re-reading

### RTT measurement to Loss and re-ordering

* Out of Order:
    * The echoed value by the receiver is that of the pkt that advanced window
      and not the new out-of-order pkt.
    * This results in the sender calculating a larger RTT. (as the in-order pkt
      was the older pkt and its TSVal was the one echoed in the fast-retrans ack)
    * This is beneficial -- as the sender can know pkts are getting re-ordered
        and not lost with the increated RTO.
* Successful Retrans
    * When the gap is closed, the TSVal is that of the just-arrived segment.
        * Note that this can be the older TSVal.
    * This also helps in making the RTO appear longer, which is good.
* Longer RTO make the sender hold on to retransmisstion in an Out-of-order case.

* Timestamp should increase by 1 atleast for 1 RTT!
* Timestamp shouldn't increase faster than by 1 for 59ns, as then it would wrap
  before IP discards a pkt.

### Timer Based Retransmission

* Note that on a healthy connection, RTO timer never expires. However, it is
  continuosly armed and reset'ed on every data-trans and ack.
* When a RTO timeout happens, TCP reacts very cautiously
    * It reduces its sender window (congestion control)
    * It introdcued an exponential backoff factor to the RTO, till a successful
      is received, at which point the backoff factor is reset to 1.
* RTO based retransmission is under-utilization of the network - as the RTO
  timer is higher than the actual RTT.
* The RTO timer based restransmission is usually a last-resort attempt to keep the
  connection going


### Fast Retrasmit

* Not timer based. Based on feedback from receiver. Hence a efficient
  retransmission strategy
* Duplicate Ack:
    *  When an OOO packet is received, the receiver generates an ACK immediately
    *  If SACK is supported, then it also has SACK block(s)
* Duplicate ACK can be due to real loss or re-order.
* So sender waits for a dupthresh(typically 3) before retransmitting
* In addition to retrasmitting pkts congestion control procedures are undertaken
  as well.

* When a fast-retransmission is done, TCP notes the highest seq number sent
  so far before the retransmission. Thi is called the recovery point.
* TCP is said to be in recovery until a ACK for the recovery point arrives.
* ACK that advances window, but not yet catching up recovery point are called
  partial-ACKs.
* On partial-ACKs TCP immediately transmits next segments.
* This partial-ACK is part of New-Reno. Before that, any acceptable ack will
  bring out of recovery.

### Retransmission with Selective ACK

* Gap between ACK and the in-window received pkts are called holes
* There are 3 or 4 sack-blocks.
* SACK is generated as soon a out of sequence is detected. It can be due to loss
  or out-of-order arrival.
* 

# Question:

* How to trigger a RST to a connection
* We explained out-of-order case, but how about Loss case?
