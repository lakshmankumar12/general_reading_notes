# OVS

## Links

* Commands are available in nixPowerToolNotes/networking

# Design

```
    Controller
    -------------------
    ovsdb-server   ovs-vswitchd
    -------------------
    Kernel-Datapath
```

* ovsdb-server
    * has long-standing configuration like information
* ovs-vswitchd
    * listens to flow updates from controller
    * Is in the data-path for first-pkts
* Kernel datapath
    * handles pkts.
    * If no match found, its sent to ovs-vswitchd, that decides and injects the flow-entry into kernel.

## ctls

* ovs-vsctl
    * manages the ovsdb-server
* ovs-appctl
    * sends cmds to the ovs-vswitchd
* ovs-ofctl
    * works with openflow protocol
* ovs-dpctl
    * kernel config module

## General design

* Actions
    * Instructions from ovs-vswitchd on how to handle a pkt.
    * Pkt modification, sampling, drop, etc..
* Open-vswitchd receives openflow tables from controller

## Caching

* Microflow cache
    * single flow cache
        * Eg: 5-tuple, HW-Addr-match, switch-ingress-port
* Megaflow cache
    * Aggregate cache

# OpenFlow

OpenFlow allows a controller to add, remove, update, monitor, and obtain
statistics on flow tables and their flows, as well as to divert selected
packets to the controller and to inject packets from the controller into
the switch.

# Links

https://aptira.com/openstack-rules-how-openvswitch-works-inside-openstack/

Papers: Packet classification through tuple based search
