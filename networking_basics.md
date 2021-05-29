Simple networking facts
========================

For Routing (BGP, iptables -- refer to routing.asciidoc)

# Ethernet

* MAC address is 6 octets

# List of protocols

DataLink
ARP, RARP
IPv4/ICMP, IPv6/ICMPv6,  IGMP (Group management, used for IPv4 for multicasting)
TCP, SCTP, UDP


# IP

* 4 octets
* Networking/Host part
* Broadcast is when host-part is all-ones.
* ICMP - Internet Control Message Protocol

* IPv4 header
  * 20 bytes (It has options)
  * 32 bit address

* IPv6 header
  * 40 bytes (It has extensions)
  * 128 bit addreses (16 octets)

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Important ip protocol numbers

1   - ICMP
2   - IGMP
6   - TCP
17  - UDP
47  - GRE



# UDP

* prot number = 17
* simple, unreliable, datagram protocol

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   .... data ....                                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


# TCP

* prot number = 6

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   .... data ....                                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


* sophisticated, reliable byte-stream protocol

* Conection Unique by (src-ip,src-port,dest-ip,dest-port)
* SYN can be responded by RST (then connect returns REFUSED), typically when nobody is listening on the port.

* TCP contains algos to measure RTT and know how long to wait for a ACK
* TCP provides flow-control . The receiver can tell how much its willing to get. This is the receive buffer.
  This is the amount of data it has received from peer, but not yet collect by local app.

## On sequence numbers and acknowledgements

* Seq number is one for every byte. SYN, FIN consume a seq of their own.
* The Seq number beings at a random number. Usually we talk of relative seq number, where 0 is the first. This is that of SYN.
* The Seq number in a pkt is that of the first byte in the packet (if present or would have been there).
* Ack of the first-SYN will always be 0. (We dont have the server's numbers yet)
* Ack from SYN-ACK is also usually give relatively. The ACK is one more than the last received pkt.

## Tcp's sliding window

* Ack represents the next byte number receiver is expecing
* Receiver window is how much the sender can send beyong the last ack.
    * For a client, the SO_RCVBUF socket option must be set before calling connect.
    * For a server, the SO_RCVBUF socket option must be set for the listening socket before calling listen.

From fig 20.4 stevens:

```
                        <-----offered window adv by recver->
                                         <--usable window-->

                        +----------------+-----------------+
       1      2     3   | 4     5     6  |  7     8     9  | 10    11     ...
                        +----------------+-----------------+

       --sent & ---->   <---sent but ---> <-can send asap--><---cant send ---
          acked             not acked                           until window moves
```




## SYN-options

* MSS - each end advertises its MSS to its peer.
       (My Q: But what if the intermediate MTU is lesser?
        My A: TCP will begin with the MTU of the recipient and use a DF bit.
              Thus it will send at the path MTU.)
* Window scale Option.
    Each SYN contains the window-size. Since this is originally 65K, its too small.
    With scale option its now scalled by 14, enabling upto 1G.
* Timestamp:

## State-Diagram

```
                       CLOSED---------------+                                                       ;
                         |                  |                                                       ;
                  (serv-passive-open)       |                                                       ;
                         |             (client-opn/send-syn)                                        ;
                         V                  |                                                       ;
                       LISTEN               |                                                       ;
                                            |                                                       ;
        (syn-rcvd/send-synack)              V                                                       ;
          SYN_RCVD                      SYN_SENT                                                    ;
                                                                                                    ;
                                                                                                    ;
                      ESTABLISHED         CLOSE_WAIT  (recv-FIN/send-ACK)                           ;
                                                                                                    ;
                                                                                                    ;
                                           LAST_ACK   (app-closes/send-FIN)                         ;
      FIN_WAIT    CLOSING                                                                           ;
                                                                                                    ;
      FIN_WAIT_2  TIME_WAIT                                                                         ;
                                                                                                    ;
                                                                                                    ;
                                                                                                    ;
```

* Note that when we get FIN, we immediately close. However, if we did close [and got FIN too] we wait
  on TIME_WAIT. This is because only one end needs to wait. Thus its chosen for the first intiator
  of the close to wait on this state. [While the other party isn't waiting, the connection per se
  isnt possible, as still one party is holding the 4 tuple]
* Timewait state is for 2MSL -- Max Segment LifeTime is the time spent by a pkt in the network
  from client to server.

## PUSH and URG flags in TCP

* PSH
    * sender intimates local TCP, to not wait for further data from application and transmit whatever is
      in its send-buffer out. TCP then sets this flag and sends whatever it has.
    * At receiver, when a Pkt with this flag is set, the TCP immediately informs it app with that data.
      Useful for RT-app exchanges like telnet, where each keystroke should be immediately tranferred.
    * Socket library doesn't provide a way to set this flag at send side. Kernel chooses to set this itself.

* URG
    * This is out of band data. The urge pointer in tcp demarcates the URG-data from normal data. Byte 1
      till urg-pointer is OOB data and rest are normal
    * MSG_OOB is the socket interface to set it.


> NOTE
>     Refer to tcp_learning.md for RFC notes.

## Delayed ack

* Just dont send ack right away, but wait for 200ms and see if there is local data to be sent. This way
  ack can be piggy backed on the data.
* Host requirements rfc states that ack should be sent utmost within 500ms.

## Nagle Algorithm

* The algo is very simple. At a given point in time, only one outstanding tinygram can be sent
  w/o an ack.
* This algo is naturally self-clocking. It accumulates lots of tiny grams in case of delaying
  networks, but kind of becomes non-felt in fast connections.
* Sometimes, this needs to switched off (like in X-window systems, where each tiny mouse-adjustment
  needs to be send for quick feedback). TCP_NODELAY options switches this off.
*

## slow start

(Read this in sctp first)
* In addition to receiver window, which is imposed by receiver, send maintains its own congestion
  window
* Initially its set to 1 segment-size. When an ack is receivd, cwnd is incrased by one.

## PAWS

Protection Against Wrapped Seq Number.

* If the bandwidth (transfer rate) is high, the possibility of re-use of sequence number
  is high on the same connection. For eg, in a 1Gbps connection, seqence number will be
  reused every 17s!

# SCTP

Reading from : http://www.slideshare.net/PeterREgli/overview-of-sctp-transport-protocol

* reliable
* message boundaries
* multiple-streamed, provides a way to minimize head-of-line blocking
    * sequenced delivery
* transport level support for multihoming
    * preforms HeartBeats to check liveliness of every PATH.
* DOS protection
* Fragmentation
* flow-control/congestion-avoidance

* Call associations
* Chunks (Sctp's own ctrl chunks and user-data chunks)
* Bundling -> putting multiple chunks in the same pkt.
* Endpoint -> hosts participating in an association
* Streams -> one uni-directional logical channel transporting application-data
    * sctp-ctrl-chunks (like init,init-ack are stream agnostic)
    * Has multiiple streams, each with its own stream identifier
    * app has to use 2 streams in each dir if they need full-duplex communication

* Like RST for TCP, ABORT is sent for SCTP when nobody is listening on the prot.
* no halfway-close like in tcp. Whoever initiates shutdown will follow 3 way close.
  It possible for both sides to initiate shutdown. Then both sides follow same 3way
  seq (as well responding to peer)
* Every Data has a unique TSN (global for the assoc)

## header

```
src-prt | dst-prt
verification-tag
checksum
chunk1
..
chunkN
```

### chunk

```
type| flags | len
data
```

## Data-chunk:

```
                              Flags
type=0 | UBE | CL             U - Unordered Data (Seq num is ignored and data prsented immdly)
TSN                           B - Beginnning of Fragment
StreamID| StreamSeqNum        E - End of Fragment
Upper-Prot-Id
Data
```

## Some chunk types

DATA, INIT, INIT_ACK, COOKIE_ECHO, COOKIE_ACK, SACK, HB, HB_ACK, ABORT, SHUTDOWN, SHUTDOWN_ACK, SHUTDOWN_COMP


### State-Diag for assoc-setup

```
client: CLOSED -> COOKIE_WAIT  -> COOKIE_ECHOED -> ESTABLISHED
server: CLOSED -> ESTABLISHED
```

* On a simul open on both sides, both sides follow client-style.

```
    ESTABLISHED

SHUTDOWN        SHUTDOWN-RECEIVED
PENDING

SHUTDOWN        SHUTDOWN_ACK SENT
SENT


       CLOSED
```

## Stuff in INIT

* Initial TSN
* a_rwnd

## Multiple-path
(Read more on this)

* SCTP has multiple IPs. However, one IP-IP is designated as primary path.
  Unless this is down, SCTP doesn't switch to secondary pathss.

## Fragmentation

* Reassembly MUST be supported. Sender fragments. If sender side support
  isn't available, then a big pkt should be rejected to upper layer.
* Each fragment has B/E bit set appropriately to 10(begin), 00(middle), 01(end)
* Each fragment has a separate TSN, but the same SSN (stream seq num). This
  is how fragments are associated to the same datagram.

(How is path MTU learnt?)

## Flow control

* Like TCP there is a receiver window
    * rwnd -> Actual receive window size at receiver. Default is 1500
    * a_rwnd -> Advertized rwnd. Value sent by sender. Sender stops sending data
      when it gets a a_rwnd == 0. It cant' send more than a_rwnd.
    * cwnd -> congestion window. Amount of data in flight(sent but not acked)
        must not exceed cwnd
* If a receiver has sent a a_rwnd(0), and a subsequent SACK with non-zero a_rwnd,
  is lost, then sender is blocked forever (as only SACK carries a_rwnd). To
  overcome this dead-loack, the sender sends zero-window-probes after RTO is
  elapsed and a_rwnd is still 0 and there are no outstanding SACKs, and if
  cwnd is not 0.
* Delayed-ack. Send ack for every other data, no later than 200ms

## Bundling policy

* Bundle size shuldnt exceed associtiation path MTU.
* bundle SACKs with highest prio.
* after SACK, if a_rwnd and cwnd permit, bundle as many retrans-DATA chunks
* Then bundle new DATA chunks

# GRE

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |C|R|K|S|s|Recur|A|Flags  |Ver  |          Prot Type            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | Key payload len               |     Key CallID                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Seq Num (optional)                                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Ack Num (Optional)                                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

C        - Checksum bit
R        - Routing bit
K 
S
s
Recur
A
```



# General algorithms

Nice documentation in:
* https://intronetworks.cs.luc.edu/1/html/tcp.html
* http://squidarth.com/rc/programming/networking/2018/07/18/intro-congestion.html

## Silly Window Syndrom

* SCTP has it. RFC 1122. Avoid advertising small a_rwnd and sender stop sending
  small pkts. This will result in poor connection throughput.

## Slow start

* In RFC 2001 (Stevens)
* While we know the receive window size, we still dont know about the network
  between sender and receiver. So, what if routers dont have enough queue capacity.
  So, Instead of just blasting pkts to the advertised window-size, we start slow
  and then catup to rwnd.
* Adds another window to sender tcp - cwnd. cwnd is initially set to 1 Segment
  size. On every ACK, cwnd is increased by one segment. senders sends the
  minimum of rwnd,cwnd. This ensure, we start with one outstanding segment and
  gradually increase till rwnd.
* Note that the actual increase is liner on every ack, but in effect it will
  increase exponentially, as initially 1 outstanding, then 2 outstanding, then
  we get 2 acks, so we send 3 outstanding etc.. But if the receiver clubs ACKs
  then the increase is not really exponential

* This site has a good graph on how slow-start and congestion-avoidance interact.
  http://www.mathcs.emory.edu/~cheung/Courses/558a/Syllabus/6-transport/TCP.html
* Another slide on these:
  https://www.slideshare.net/engineerrd/tcp-congestion-avoidance<Paste>

## Congestion Avoidance

* Congestion happens when a big PIPE connects to a smaller pipe or the o/p of
  a router doesn't match sum of inputs. There are two indications of packet 
  loss:  a timeout occurring and the receipt of duplicate ACKs.
* While congestion avoidance and Slow-start are indepedant in theory, their
  implementation is intertwined.
* We have one more variable - ssthred (slow start threshold)

* Initially ssthres = 65535 (max-window size?)
* When congestion occurs i.e Either of timeout or dup ack
   set ssthres = half of curr-win(min of a_rwnd, cwnd)
                 But always min of 2*seg-size
  If it was timeout
   set cwnd = 1 segment (This is when we really stop sending much)
* Now on every ACK, we keep increasing cwnd (subject to no loss/timeout)
* Now, whether we slow-start (exponentially or linearly) depends on following:
  if `cwnd <= ssthres`, its slow-start (normal case)
  otherwise congestion avoidance.
* If its slow-start, we increase cwnd for every ACK. But in congestion
  avoidance, we increase only `segsize*segsize/cwnd` each time an ACK is recvd.
  This is linear. Basially its saying increase cwnd by MSS after ACKs for cwnd
  worth of data are received.

## Fast-restranmit

* Normally a receiver sends a ACK immdly out in case it sees a out-of-order.
* This could have happend because of a re-order or a lost segment. If re-order
  nothing is reqd from sender.
* So sender when it sees a duplicat-ack, doesn't send out immdly, but rather
  waits to see 2 or 3 dup-ack and then sends a retrans before the timeout.

## Fast-recovery



# MP-TCP

* Both parties need to support this (Confirm..)


# ICMP

* ICMP echo-req/rsp pkts have both a ID and seq-number. The ID is useful to enable NAT'ing - like port numbers for UDP.

# DHCP

4 main packets:

This is popularly known as DORA

* DHCP-DISCOVER
    -- Src-IP: `<??>`  Dst-IP: 255.255.255.255
* DHCP-OFFER
    -- Src-IP: `<dhcp-server>` Dst-IP: `<??>`
    -- contains the offered IP.
* DHCP-REQUEST
* DHCP-ACK

In Ethereal, these are UDP-packets with port 67 - referred as BOOTP protocol

## Relaying DHCP

* The important field here is the Gateway-IP. The relay-agent puts in its gateway ip/Relay-agent-ip and forwards the broadcast as a unicast to the server.
    * The gw-ip help the server in identifying which subnet to lease from.
    * The reply goes back to gw-ip


# SNMP

mib-2 is a short for OID: .1.3.6.1.2.1
iso(1) identified-organization(3) dod(6) internet(1) mgmt(2) mib-2(1)

# Well known Ports

FTP        | 21 - control (TCP)  Is there a UDP??
           | 20 - data
SSH        | 22 - TCP.
Telnet     | 23
SMTP       | 25
DNS        | 53
DHCPv4srv  | 67
DHCPv4clt  | 68
TFTP       | 69
HTTP       | 80
POP3       | 110
NTP        | 123
NetBios    | 137
IMAP       | 143
SNMP       | 161
HTTPS      | 443
DHCPv6srv  | 546
DHCPv6clt  | 547

* Note 1-1023 are considered well-known ports
* 1024-65535 are called registered ports
    * ephemeral ports are another name

# Wifi Networks

* SSIDs are case-sensitive
* 802.11 b/g -> the 2.5 band
* 802.11 n/ac -> the 5Ghz band
* Auth types
    * open
        -- none at wifi-level
    * PSK
    * EAP
* Encryption
    * Wired Equivalency Protocol (WEP)
        -- static key
    * Wi-Fi Protected Access (WPA)
        -- dynamic keys
        -- WPA-PSK2 pre-shared key

# Cabling types / slots

Search: cable

* SFP (small form-factor pluggable)
* Enhanced high-speed wan interface card (EHWIC slots)
* RJ-45 (cat-5, UTP(unshielded twisted pair))
    * T568A or T568B -- std for the pairing on the 8 pins
* RJ-11
* DB-9
* AUX port - Used for remote management of the router using a dial-up telephone line and modem.

# Cisco led cues

## Port Status, or STAT, the Default Port Mode:

* Off: No link, or port was administratively shut down.
* Green: Link present.
* Blinking green: Port is transmitting or receiving data.
* Alternating green-amber: Link fault. Error frames can affect connectivity, and errors such as excessive collisions, CRC errors, and alignment and jabber errors are monitored for a link-fault indication.
* Amber: Port is blocked by Spanning Tree Protocol (STP) and is not forwarding data.
* Blinking amber: Port is blocked by STP but continues to transmit and receive inter-switch information messages.

## Speed - For 10/100/1000 ports

* Off: Port is operating at 10 Mbps
* Green: Port is operating at 100 Mbps.
* Blinking green: Port is operating at 1000 Mbps.

# What all happen when you type a URL In a browser?


In an extremely rough and simplified sketch, assuming the simplest possible
HTTP request, no proxies, IPv4 and no problems in any step:

1. browser checks cache; if requested object is in cache and is fresh, skip to #9
2. browser asks OS for server's IP address
3. OS makes a DNS lookup and replies the IP address to the browser
4. browser opens a TCP connection to server (this step is much more complex with HTTPS)
5. browser sends the HTTP request through TCP connection
6. browser receives HTTP response and may close the TCP connection, or reuse it for another request
7. browser checks if the response is a redirect or a conditional response (3xx
   result status codes), authorization request (401), error (4xx and 5xx),
   etc.; these are handled differently from normal responses (2xx)
8. if cacheable, response is stored in cache
9. browser decodes response (e.g. if it's gzipped)
10. browser determines what to do with response (e.g. is it a HTML page, is it an image, is it a sound clip?)
11.  browser renders response, or offers a download dialog for unrecognized types

Again, discussion of each of these points have filled countless pages; take
this only as a short summary. Also, there are many other things happening in
parallel to this (processing typed-in address, speculative prefetching, adding
page to browser history, displaying progress to user, notifying plugins and
extensions, rendering the page while it's downloading, pipelining,
connection tracking for keep-alive, checking for malicious content etc.) -
and the whole operation gets an order of magnitude more complex with HTTPS
(certificates and ciphers and pinning, oh my!).


# Generally in a host/router

```
            IN-FROM-WIRE
               |
               |
               v
            LOCAL IP   --> FORWARD -> WIRE
               |
               |
               v
            PROCESS
(to draw fully)
```

# Socket programming call sequence

## socket
```
  int socket(int domain, int type, int protocol);
    domain - AF_INET, AF_INET6
    type - SOCK_STREAM, SOCK_DGRAM, SOCK_SEQPACKET, SOCK_RAW (SOCK_NONBLOCK, SOCK_CLOEXEC may also be bit-OREed)
    protocol - IPPROTO_TCP/IPPROTO_UCP/IPPROTO_SCTP
    INET/6, STREAM - TCP/SCTP
    INET/6, DGRAM  - UDP
    INET/6, SEQPACKET  - SCTP
    INET/6, RAW - IPv4,IPv6
    LOCAL, STREAM/DGRAM/SEQPACKET  - Yes
    ROUTE, RAW - Yes   (Kernel routing table)
    KEY, RAW - Yes     (Cryptography)
```
## bind
```
  int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
    * Used to bind the local addr to listend in case of servers.
    * Used for clients? Not needed the kernel picks a ephemeral port and some local ip.
      If you desire to pick one, u can use bind
## listen
```
  int listen(int sockfd, int backlog);
```
    * returns immdly. backlock is the number of pending connections.
## accept
```
  int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  int accept4(int sockfd, struct sockaddr *addr, socklen_t *addrlen, int flags);
```
    * returns the fd of the new connection. The addr is a o/p field that stores the addr of the remote party
    * may block or not block depending on connection availble.
    * in case of tcp, only fully handshaked connections are notified in accept.
    * flags are NON_BLOCK/CLOSONEXEC
## connect
```
  int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
    * connects to a remote server mentioned.
    * For TCP/SCTP initiates the association. No response to handshacke, it returns timeout
    * For UDP, creates the default dest addr. (can be changed by another call to connect)
    * For TCP, connecting on a connected socket generates a RST (linux behavior!)
        * This is one trick to generate a RST by application on a socket
## shutdown
```
  int shutdown(int sockfd, int how);
```
    * useful for half-close (either cloase reading or writing)
## close
```
  int close(int fd);
```
    * closes the fd, w.r.t this process.
    * Upon all process counts to this fd comign to 0,
      The kernel however will flush all to-send data and send FIN
## send
```
  ssize_t send(int sockfd, const void *buf, size_t len, int flags);
  ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
  ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
  int sendmmsg(int sockfd, struct mmsghdr *msgvec, unsigned int vlen, unsigned int flags);
```
    * send shoudl be called only on conneced sockets where dest is unknown. In connected sockets, dst if passed is ingored or
      EISCONN returned
## receive
```
  ssize_t recv(int sockfd, void *buf, size_t len, int flags);
  ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
  ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
  int recvmmsg(int sockfd, struct mmsghdr *msgvec, unsigned int vlen, unsigned int flags, struct timespec *timeout);
```
    * recvmmsg extends both on multi-messages + a extra timeout.
    * See readv for how iovecs are used.

```
Server:
  socket, bind, listen, accept,   recv/send, close

Client:
  socket, optional-bind, connect,   recv/send, close
```


* tcp client, should add the just connect-issued socket to the write list to test for HS complete (to confirm)
* tcp server, should add the listen-fd to the READ-list to check for incoming connections and issuing accept.

# Some libraries that bypass kernel network processing

netmap(BSD), DPDK

# Token Bucket and Leaky Bucket

* Token bucket is a traffic policer
* Leaky bucket is a traffic shaper


At a high level, a traffic policer limits traffic to a bandwidth. It either
passes or drops. A traffic shaper, holds pkts in a buffer, and transmits it
slowly.

* In general, you can only control the traffic you send, not what you
  receive.


Virus protection
Spyware protection
Spam blockers
Popup blockers
Hardware Firewalls
Software Firewalls
