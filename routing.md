# Routing Protocols

## Hierarchy

* Interior Gateway Protocols
    * __OSFP__
        * link-state.
        * Considers cost
    * __RIP__
        * distance vector
        * Hop Count
    * __EIGRP__
        * distance vector
        * Bandwidth, Delay, Load and Reliability (also called the K-values)
* Exterior Gateway Protocols
    * __BGP__
        * path vector

## Types of Routing Protocol

Nice article on the difference between link-state, distance-vector and
path-vector : http://www.informit.com/articles/article.aspx?p=331613&seqNum=2

* Distance-Vector
    * Each router just advertises all routed reachable through it. These are
      both direct routes and that it got from others. Upon receipt of an
      advertisement, a router updates its routing table and passes on this in
      its future advertisement.
    * Example protocols are RIP, EIGRP

* Path-Vector
    * Here again, each router updates its summarized info. However, in addition
      to giving a route and its metric(the distance), it also gives the path to
      that route. This helps in a receiving router to detect loops before it
      adds a path to its table.
    * Eg protocols are BGP

Figuratively, both distance and path-vector is update-by-rumor, as each router
trusts the advertisement of routes themselves from their peers

* Link state
    * Here each router just updates their directly connected link information
      to every other router (both directly connected and not). Thus every
      router gets to develop the full network topology for itself. Each router
      then builds up the shortest path using some SPF algo like Djikstra. Eg
      are OSPF, IS-IS.


# BGP

[Good Link](https://www.youtube.com/watch?v=ZucnfoJiFr8)

* https://www.kentik.com/blog/bgp-routing-tutorial-series-part-1/

* BGP is optimzied for scalability, not convergence time
* BGP had advanced filtering rules, foreg, you can say i dont want routes using AS-xyz/
* uses AS-path list to avoid loops
* number of AS-hops as a universal metric

* Only 4 message types - OPEN/KEEPALIVE/UPDATE/NOTIFICATION
* Only 6 states
* many filtering options

* Uses TCP/179. Explicit router to router connection. No multicast/broadcast. Routers are known by config. No dynamic learning
* remote pkts are accepted only from configured peers

* Open message
    * BGP version, sender AS number, Hold time (def: 180s), highest loopback, optinal params
    * Done like open, send notifications

* Keep Alive
    * 1/3rd of hold time, 60s def

* Update
    * NLRI(Network Layer Reacheability Info) - just a complicated way of saing ip/prefix
    * Path updates
    * Withdrawn routes

* Notification
    * terminates/rejects a BGP
    * Can include error message

* Neighbour states
    * IDLE - start up or some bgp error occured
    * CONNECT - Initiated a TCP connection
    * ACTIVE - Repeated attempting a connection
    * OPEN-SENT - TCP established, sent open message
    * OPEN-CONFIRM - received open back
    * ESTABLISHED - received a KA or update

* EBGP session and IBGP sessions
    * Typically EBGP runs on a physical IP as we want EBGP to go down on a link-down.
    * However, IBGP typically runs on loopback, as there may be other paths to the peer.
    * IBGP peers neednt be directly connected
    * IBGP neighbors can be acroos hops (TTL of 255 is used), however, EBGP neighbors should be next-hop (TTL of 1 is used)
    * In cisco config, we dont explicity distinguish IBGP/EBGP. We configure remote-peer -ip and their AS. Sam AS => IBGP, diff AS => EBGP

* Attributes
    * Each advertised route has attributes
    * There are about 13/14 attributes
    * well-known attributes - defined by protocol, understood by all routers
    * 2 types of well-known: madatory and discretionary (ma not be included in every update)
    * Optional - Transitive (pass it on even if you dont understand) / Non-transitive (drop it if you dont undertand)

* Well-Known Madatory
    * Origin Attribute
        * IGP - route learnt from IGP (configured routes)
        * EGP - learnt from the obsolte protocol EGP
        * Incomplete - real origin not known
    * AS-Path
        * List of AS that the route passed through
    * Next-Hop
        * IP address to forward to

* Well-known discretionary
    * Local preference
        * Route prioritization. Higher => better, Default 100
    * Atomic Agreegate -  route was summarized

* Transitive Option
    * Aggregator: Identifies AS/router that did the aggregatin
    * Community: Message in the udpate

* Non-Transitive
    * MED - Multi Exit Discriminator (didnt understand that)

# IP Tables

* iptables is the command, netfilter is the kernel module. iptables work at layer-3.
* TABLES by function. each table has CHAINS of RULES. rules consist of MATCHES and TARGETS
    ```
    TABLES --> (CHAINS of (RULES --> MATCHES and TARGETS))
    ```
  Actually, its much more convenient to see as Chains --> (Tables), as pkt processing order
  is chain-first and then tables that have that chain.
* It is however only partially correct that chains belongs to tables. Neither belongs to the other.
  Tables represents types of processing, chains represent hook points.
* Eg command:
    ```
    iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.3:8080
    -t nat                            Operate on the nat table...
    -A PREROUTING                     ... by appending the following rule to its PREROUTING chain.
    -i eth1                           Match packets coming in on the eth1 network interface...
    -p tcp                            ... that use the tcp (TCP/IP) protocol
    --dport 80                        ... and are intended for local port 80.
    -j DNAT                           Jump to the DNAT target...
    --to-destination 192.168.1.3:8080 ... and change the destination address to 192.168.1.3 and destination port to 8080.
    ```

* 5 hook points - PREROUTING, INPUT, FORWARD, POSTROUTING, OUTPUT.
    * built in chains are attached to these hook points. (Chains == hook points in a way)
    * we add a sequence of rules for each hook point. Each rule may affect or monitor a packet.
    * While iptables call it target, I find it convenient to read that as Action-to-take.

## Typical chain flow

    ```
    R-D: Routing-Decision

    +--------+    +--------+        +---+      +--------+                   +--------+
    |Ntwk    |--->|PRE     |--------|R-D|----->|INPUT   |------------------>|Local   |
    |Intf    |    |ROUTING |        +---+      |        |                   |Process |
    +--------+    +--------+          |        +--------+                   +--------+
                                      v          ^
                                  +--------+     |
                                  |FORWARD |     |
                                  |        |     |
                                  +--------+     |
                                      |          |
    +--------+    +--------+          v        +---+   +--------+  +---+   +--------+
    |Ntwk    |<---|POST    |<------------------|R-D|<--|OUTPUT  |<-|R-D|---|Local   |
    |Intf    |    |ROUTING |                   +---+   |        |  +---+   |Process |
    +--------+    +--------+                           +--------+          +--------+
    ```


### Note the normal flows:

* Inbound pkts from Intf: Intf --> PREROUTING --> INPUT --> Process
* Outbound pkts to Intf:  Process --> OUTPUT --> POSTROUTING --> Intf
* Routed pkts:            Intf --> PREROUTING --> FORWARD --> POSTROUTING --> Intf
* Process-to-Process:     Process --> OUTPUT --> INPUT --> Process

### NAT Table chains

    ```
    +--------+         +--------+                       +--------+
    |Ntwk    | ------->|PRE     |---------------------->|Local   |
    |Intf    |         |ROUTING |          ^            |Process |
    +--------+         +--------+          |            +--------+
                           |               |
                           |               |
                           v               |
    +--------+         +--------+      +--------+       +--------+
    |Ntwk    |<--------|POST    |<-----|OUTPUT  |<------|Local   |
    |Intf    |         |ROUTING |      |        |       |Process |
    +--------+         +--------+      +--------+       +--------+
    ```

### Filter Table Chains

```
+--------+                      +--------+         +--------+
|Ntwk    |--------------------->|INPUT   |-------->|Local   |
|Intf    |          |           |        |         |Process |
+--------+          v           +--------+         +--------+
                +--------+           ^
                |FORWARD |           |
                |        |           |
                +--------+           |
                    |                |
+--------+          v            +--------+        +--------+
|Ntwk    |<--------------------- |OUTPUT  |<-------|Local   |
|Intf    |                       |        |        |Process |
+--------+                       +--------+        +--------+
```

### Packet Mangling Table Chains

```
+--------+         +--------+                       +--------+
|Ntwk    | ------->|PRE     |---------------------->|Local   |
|Intf    |         |ROUTING |          ^            |Process |
+--------+         +--------+          |            +--------+
                       |               |
                  +--------+           |
                  |FORWARD |           |
                  |        |           |
                  +--------+           |
                       |               |
                       v               |
+--------+         +--------+      +--------+       +--------+
|Ntwk    |<--------|POST    |<-----|OUTPUT  |<------|Local   |
|Intf    |         |ROUTING |      |        |       |Process |
+--------+         +--------+      +--------+       +--------+
```

## Hook points

FORWARD     ... that flow through a gateway computer, coming in one interface
                and going right back out another.
INPUT       ... just before they are delivered to a local process.
OUTPUT      ... just after they are generated by a local process.
POSTROUTING ... just before they leave a network interface.
PREROUTING  ... just as they arrive from a network interface (after dropping
                any packets resulting from the interface being in promiscuous
                mode and after checksum validation)

## 3 builtin tables

### nat

Used with connection tracking to redirect connections for network
address translation; typically based on source or destination addresses.
Its built-in chains are: OUTPUT, POSTROUTING, and PREROUTING.

### filter

Used to set policies for the type of traffic allowed into, through,
and out of the computer. Unless you refer to a different table
explicitly, iptables operate on chains within this table __by default__.
Its built-in chains are: FORWARD, INPUT, and OUTPUT.

### mangle

Used for specialized packet alteration, such as stripping off IP options (as
with the IPV4OPTSSTRIP target extension). Its built-in chains are:
FORWARD, INPUT, OUTPUT, POSTROUTING, and PREROUTING

## Chains

* A chain’s policy determines the fate of packets that reach the end of the chain without otherwise being sent to a specific target.
* Only the built-in targets (see Table 8) ACCEPT and DROP can be used as the policy for a built-in chain, and the default is ACCEPT.
* All user-defined chains have an implicit policy of RETURN that cannot be changed

* If you want a more complicated policy for a built-in chain or a policy other than RETURN for a user-defined chain,
    you can add a rule to the end of the chain that matches all packets, with any target you like.
* You can set the chain’s policy to DROP in case you make a mistake in your catch-all rule or
  wish to filter out traffic while you make modifications to your catch-all rule (by deleting it and re-adding it with changes)

## Packet Flow

* Packets traverse chains, and are presented to the chains’ rules one at a time in order.
* If the packet does not match the rule’s criteria, the packet moves to the next rule in the chain.
* If a packet reaches the last rule in a chain and still does not match, the chain’s policy
  (essentially the chain’s default target) is applied to it

```
Between tables order (both input/output):
mangle --> nat --> filter
```

### pkt flow from network-interface -> another-network-interface
TABLE    CHAIN
mangle   PREROUTING
nat      PREROUTING
mangle   FORWARD
filter   FORWARD
mangle   POSTROUTING
nat      POSTROUTING

### network-interface -> local-process
Table    Chain
mangle   PREROUTING
nat      PREROUTING
mangle   INPUT
filter   INPUT

### local-process -> network-interface
Table    Chain
mangle   OUTPUT
nat      OUTPUT
filter   OUTPUT
mangle   POSTROUTING
nat      POSTROUTING

### local-process -> local-process
Table    Chain
mangle   OUTPUT
nat      OUTPUT
filter   OUTPUT
filter   INPUT
mangle   INPUT

## Rules

* consists of one or more match criteria that determine which network
  packets it affects (all match options must be satisfied for the rule
  to match a packet) and a target specification that determines how
  the network packets will be affected
* system maintains a pkt & byte couner for every rule.
* Match is optional => all pkts match
* Target is also optional(!) => as-if rules doesn't exist. Just stat'ed.

## Matches

* Generic matches available with the kernel-compiled stuff
* Extenstion possible with the -m option.

### Popular matches

```
-d 1.30.3.2/32
-s 192.168.2.0/24
-i eth3
-p gre
-m udp --dport 4502
-m icmp --icmp-type 3/4
```

## Targets

* 4 built-in targets. Extension provide other targets

ACCEPT  Let the packet through to the next stage of processing.
        Stop traversing the current chain, and start at the next stage
DROP    Discontinue processing the packet completely.
        Do not check it against any other rules, chains, or tables.
        If you want to provide some feedback to the sender, use the REJECT target extension.
QUEUE   Send the packet to userspace (i.e. code not in the kernel).
        See the libipq manpage for more information.
RETURN  From a rule in a user-defined chain, discontinue processing this chain, and
        resume traversing the calling chain at the rule following the one that had this chain
        as its target.
+
        From a rule in a built-in chain, discontinue processing the packet and apply the chain’s
        policy to it.

### Popular targets

```
mangle-PREROUTING   -- This is where we mark packets, so that this mark is leveraged on routing-decision.
-j MARK --set-xmark <mark-value>

filter-INPUT
-j DROP #Discontinue processing the packet completely.
        #Do not check it against any other rules, chains, or tables.
        #If you want to provide some feedback to the sender, use the REJECT target extension.
-j ACCEPT  #Let the packet through to the next stage of processing.
           #Stop traversing the current chain, and start at the next stage
-j QUEUE   #Send the packet to userspace (i.e. code not in the kernel).
           #See the libipq manpage for more information.
-j RETURN  #From a rule in a user-defined chain, discontinue processing this chain, and
           #resume traversing the calling chain at the rule following the one that had this chain
           #as its target.

nat-OUTPUT
-j SNAT --to-source 172.18.10.206
-j DNAT --to-destination 172.18.10.230
```

## Sample application use-cases

* Packet filtering
* Accouting
* Connection tracking
* Packet mangling
* NAT
* Masquerading
* Port forwarding
* Load balancing

# Tun/Tap Interfaces

http://backreference.org/2010/03/26/tuntap-interface-tutorial/

* Software only interfaces.
  * Kernel treats them as regular interfaces. But when it time to send the data "in the wire", the kernel gives it to the user-space process.
  * User-space program receives the data using a special descriptor.
* Its either
  * Transient. Interface is gone, once the process that created it dies (even if it didn't explicitly delete the interface)
  * Persistent. Process creates and can die. Another process can re-attach and send/recv data from the interface.
* open('/dev/net/tun',O_RDWR) is the way to create a fd that represents the tun/tap interface. This is called the clone device.
    * but we are are not done yet. ioctl(fd, TUNSETIFF, &(struct ifreq*)) tell the name and mode(tun/tap) of the interface.
    * once done, we are ready. ioctl(fd, TUNSETPERSIST/TUNSETOWNER/TUNSETGROUP) are needed if u need persistency and allow user-space prog
      to attach to the interface.
* Name is optional. If not given kernel assigns the next available "tunN/tapN" name.
* It name is already present, its assumed to be a reAttach-To-Existing interface call.

## Type of interfaces

This is not a standard classfiction -- just for my understanding. To some extent this is shown in 'ip addr show' listing.

* Ethernet  - These are typically flagged as BROADCAST.  link/ether
* tuntap    - If they are tap, then they are also shows as link/ether and are shown as ether.
              If they are tun, then we see link/[65534]
* loopback  - link/loopback
* imq       - Not really a interface. Intermediary queue for traffic shaping. You create this q.
              And make use of it using iptables rules -i <dev> -j IMQ --todev 0
* gre       - link/gre
* gretap    - gre over ethernet? link/ether
* vti       - for ipsec, link/ipip

## So, when do u use them?

* This interface is used in an upside-down fashion. Typical interfaces are where the kernel gets the pkt somehow (from wire) and does routing
  and gives to application using socket-api. This interface is useful to grab all pkts from different interfaces using routes

My Question:
1. What if the given interface is already attached by another process?
2. What happens on a fork? I presume, this is not a big deal -  just like
   how a tcp socket is handled on fork. Whichever process does a read()
   first collects pkt on the socket!

# Bridges


# Links

* Traffic-Control in Linux
  http://linux-ip.net/articles/Traffic-Control-HOWTO/

* Linux Advanced Routing & Traffic Control
  http://lartc.org/howto/index.html
