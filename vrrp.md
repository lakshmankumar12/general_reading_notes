# rfc

5798

# basics

* There is a IP that is primarily owned by one router, and this
  floats to other routers when the owner is down.
* Here is a picture giving the idea
* Rtr-1 is the owner of IP-A.
    * Rtr-2 has its own IP-B. But it takes over IP-A when rtr-1 is down.

```
        +-----------+ +-----------+
        |   Rtr1    | |   Rtr2    |
        |(MR VRID=1)| |(BR VRID=1)|
        |           | |           |
VRID=1  +-----------+ +-----------+
IPvX A--------->*            *<---------IPvX B
                |            |
                |            |
----------------+------------+-----+----------+----------+----------+--
                                   ^          ^          ^          ^
                                   |          |          |          |
default rtr IPvX addrs-------> (IPvX A)   (IPvX A)   (IPvX A)   (IPvX A)
                                   |          |          |          |
                          IPvX H1->* IpvX H2->* IPvX H3->* IpvX H4->*
                                +--+--+    +--+--+    +--+--+    +--+--+
                                |  H1 |    |  H2 |    |  H3 |    |  H4 |
                                +-----+    +-----+    +--+--+    +--+--+
   Legend:
         --+---+---+-- = Ethernet, Token Ring, or FDDI
                     H = Host computer
                    MR = Master Router
                    BR = Backup Router
                    *  =  IPvX Address; X is 4 everywhere in IPv4 case
                                        X is 6 everywhere in IPv6 case
                    (IPvX) = default router for hosts
```

# terms

* VRRP protocol number is `112`
* Multicast group
    * The master keeps sending advertisement to this multicast group
    * Its `224.0.0.18`
* Virt-Mac
    * There is not just IP takeover, but its also done along with mac-takeover.
    * The grat-arp is to help switches learn the new mac mapping to ports.
* Virt Router
    * The logical router that is moving across
    * All participating real routers help to sport this virt-router to the rest of hosts
* Prio
    * 255 - for owning router. There is no way to pre-empt this
    * 100 - def for backing routers
    * 0 - spl value to relinquish master role
* Timers
    * Master-adv - time at which the master keeps sending advertisements
    * Master-down - time at which backups decide master is no longer there.

# States

* init
* master
* backup

# Election

```
if state == init:
    if startup_event:
        if priority == 255:  # owns the ip associated
            send an ADVERTISEMENT
            if ipv4:
                send gratuitous ARP
            elif ipv5:
                send unsolicted ND Neighbor Advertisement(
                            Router flag=set,
                            Solicited flag=unset,
                            Override flag=set,
                            target-address=..., linkaddr=...)
            Set Adver_Timer to Advertisement_Interval
            Transition to the {Master} state
        else:
            Set Master_Adver_Interval to Advertisement_Interval
            Set the Master_Down_Timer to Master_Down_Interval
            Transition to the {Backup} state
elif state == Backup:
    if ipv4:
        no arp response
    elif ipv4:
        no ND neighbour solicit for the virt-address
        no ND router adv for virt router
    Discard pkts to the link-layer MAC of virt-router
    Not accept pkts to the virt-ip-address
    if shutdown:
        Cancel the Master_Down_Timer
        Transition to the {Initialize} state
    if Master_Down_Timer fires:
        Send an ADVERTISEMENT
        if ipv4:
            Broadcast a gratuitous ARP request on that interface
        elif ipv6:
            Compute and join the Solicited-Node multicast address
            send ND neigh adv ...
        Set the Adver_Timer to Advertisement_Interval
        Transition to the {Master} state
    If an ADVERTISEMENT is received:
        if prio == 0:
            Set the Master_Down_Timer to Skew_Time
        else:
            If Preempt_Mode == False or prio >= local_priority:
               Set Master_Adver_Interval to Adver Interval contained in the ADVERTISEMENT
               Recompute the Master_Down_Interval
               Reset the Master_Down_Timer to Master_Down_Interval
            else:
               Discard the ADVERTISEMENT
elif state == Master:
    //Note that in the Master state, the Preempt_Mode Flag is not considered.
    if ipv4:
        MUST respond to ARP requests for the IPv4 address
    elif ipv6:
        MUST be a member of the Solicited-Node multicast
        MUST respond to ND Neighbor Solicitation message
        MUST send ND Router Advertisements for the virtual router.
        If Accept_Mode is False:
            MUST NOT drop IPv6 Neighbor Solicitations and Neighbor Advertisements.
    MUST forward packets with a destination link-layer MAC
            address equal to the virtual router MAC address.
    MUST accept packets addressed to the IPvX address(es)
          associated with the virtual router if it is the IPvX address owner
          or if Accept_Mode is True.  Otherwise, MUST NOT accept these
      packets.
    If Shutdown event:
        Cancel the Adver_Timer
        Send an ADVERTISEMENT with Priority = 0
        Transition to the {Initialize} state
    If Adver_Timer fires:
        Send an ADVERTISEMENT
        Reset the Adver_Timer to Advertisement_Interval
    If an ADVERTISEMENT is received, then:
        If the Priority in the ADVERTISEMENT is zero, then:
            Send an ADVERTISEMENT
            Reset the Adver_Timer to Advertisement_Interval
        else: // priority was non-zero
            If the Priority in the ADVERTISEMENT > local
                            or
                    priority == local && ip_addr_val > local:
                Cancel Adver_Timer
                Set Master_Adver_Interval to Adver Interval contained in the ADVERTISEMENT
                Recompute the Skew_Time
                Recompute the Master_Down_Interval
                Set Master_Down_Timer to Master_Down_Interval
                Transition to the {Backup} state
            else:
               Discard ADVERTISEMENT
```
