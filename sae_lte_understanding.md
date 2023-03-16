SAE and LTE General Understanding
==================================
:toc:

# List of interfaces

Most common Ifcs: S1, S11, S5, SGi

* S1-C -  Control plane from Enb to MME
* S1-U -  Data plane from Enb to SGW
* S2   -
* S2a  -  Trusted non-3GPP to PGW (can be PMIP or TWAG(Gtp))
* S2b  -  Non-Trusted non-3GPP to PGW (ePDG)
* S3   -  S4-SGSN to MME
* S4   -  S4-SGSN to SGW
* S5   -  SGW to PGW
* S6   -
* S6a  -  MME to HSS
* S6b  -  PGW to AAA
* S6d  -  S4-SGSN to HSS
* S7   -  NOT_DEFINED
* S8   -  inter-plmn S5 (SGW-PGW)
* S9   -  home PCRF to visited PCRF
* S10  -  inter MME
* S11  -  MME to SGW
* S12  -  Direct Tunnel RNC->SGW
* S13  -  MME to EIR
* S14  -  NOT_DEFINED
* S15  -  NOT_DEFINED
* S16  -  S4-SGSN to S4-SGSN
* S17  -  MME to UCMF
* S101 -  eHRPD to MME (Pmip?)
* S103 -  HSGW to SGW (Pmip?)
* SGi  -  PGW to Packet-Data-Network

* X2 - eNodeb to eNode

Rx  -  PCRF to PDN-applications (like PCSCF)
Gx  -  PCRF to PGW
Gxa -  HSGW(pmip) to PCRF
Gxc -  PCRF to SGW
Gy  -  online  charging to PGW
Gz  -  offline charging to PGW

SWu                   - UE to ePDG
SWm  (diam)           - ePDG to AAA(untrusted)
SWw  (radius)
STa  (diam or radius) - HSGW(Pmip) to AAA
SWx                   - AAA to HSS

## UMTS

Ga - {GGSN,SGSN,SGW,PGW} -> CGF
Gb - SGSN to BSS
Gc - GGSN to HLR
Gd - SMSC to SGSN
Gf - SGSN to EIR
Gn/Gp - SGSN to GGSN
Gr - SGSN to HLR
Gs - SGSN to MSC/VLR
Iu - RNC to SGSN
Gi - GGSN to IP-Network

# Spec list

23.401 - 4G general architecture

24.301 - Nas Signalling
29.274 - GTP

36.300 - eNodeb

23.060 - 3G general architecture
24.008 - GMM
25.413 - RANAP
29.060 - GTPv1

23.501 - 5G
23.502 - 5G NGAP


# IDs

* PLMN-ID   = MCC + MNC
* MMEI      = MMEGI + MMEC
    * MME-group-identifier, 16 bits
    * MME-code (one mme withing the group), 8 bits
* GUMMEI    = PLMN-ID + MMEI
    * Globally unique MMEI
* TAI       = PLMN-ID + TAC
* EGCI      = PLMN-ID + ECI
    * EUTRAN (global) cell identifier
* S-TMSI    = MMEC + M-TMSI
    * S-TMSI, identifies TMSI within pool-area
    * M-TMSI, identifies TMSI within MME
* GUTI      = PLMN-ID + MMEGI + S-TMSI
    * Globally unique Temporary Identifier

Has good info on IDS - https://www.prodevelopertutorial.com/lte-chapter-6-identifiers-in-lte/


# Policy and Bearers

## Bearer

* One logical connection between UE and PGW. It traveses
  ```
  UE ------ Enodeb ---- SGW  ----- PGW
     Radio         S1        S5/S8
     Bearer        Bearer    Bearer
     <---E-RAB--------->
     <---EPS-Bearer--------------->

  ```
* Is uniquely identfied by (QCI,ARP)
    * QCI - Quality of Class Identifier
        * 1 to 9.
        * Each number maps to a specification defined preset
          value of pkt-loss,pkt-delay and usecase.
    * ARP - allocation and retention priority
        * a number from 1 to 15 , 1 highest prio

## PGW and PCRF

* The PGW and PCRF maintain session-level contexts.
* Each session is identified at PGW by (IMSI,APN-requested,Access-type) and so is it at PCRF
* The following is the hierarchy of information exchanged between PGW and PCRF
    * Session-level
        * Default-Bearer's QCI/ARP
        * AMBR for session
    * Bunch of Rules. Each rule has the following
        * Match criteria
        * Action.
            This could be just gating/drop or apply some Qos. Qos is specified in terms of a QCI/ARP. Another action
            is to classify this traffic into a (Rating-Grounp,service-id) for charging.
* The PCRF and PGW are in sync about the current session that is ongoing, and
  the rules that are applied/in-progress for that session.
* The PCRF may at any point add a new rule to a session, delete a rule or edit a rule.
* Note that there is no notion of bearer per se, between PCRF and PGW

## Understanding a Rule

* Rules have priority amongst them. So Rules with higher priority are evaluated first.
* Within a rule there are multiple match-criteria. These are natually ordered in the order
  in which PCRF specified them.
    For eg, Say Rule R1 has M1,M2,M3 and Rule R2 has M4,M5  and R1 has highter priority, then a
    PGW will apply M1,M2,M3,M4 and M5 in that order


## PGW job of mapping Rules to Bearers

* PGW basically decides to create a new bearer if any new rule has a different (QCI/ARP) from
  other existing rules. This is because each bearer has a unique (QCI/ARP) for itself.

## Open Questions?

* Is APN-AMBR really session level?
* Is there a UE-AMBR? Does SGW apply it then?
* Does matches have bit-rates apart from QCI/ARP. How does that work?

# Security

* Very good break-down in https://www.prodevelopertutorial.com/lte-chapter-10-lte-security/

# UE

## Info stored by UE

* USIM
    * Network Operator's PLMN list
    * Subscription Information
    * Access Barring
* Stored Information
    * Most recently used frequency band
    * PLMN
    * Tracking Area Code
    * Cell ID
    * S-TMSI
    * InterRAT Frequency Band
* Information that UE needs to get
    * Frequency and Timing Synchronization info
    * System Bandwidth
    * Number of MIMO Antennas
    * Identities
        * (C-RNTI, Physical Cell ID, Tracking Area Code)
        * Network PLMN
        * Signaling & Traffic Radio Resouce
        * RACH_ROOT_SEQUENCE
        * PRACH Config.

## Power-On Sequence

* UE is Off
* Power On UE
* Frequency Search
* Time and Frame Synchronization
    * In this process, PSS and SSS will be decoded as well.
* PCI (Physical Cell ID) detection
* MIB decoding
    * UE can figure out System Bandwidth and Transmission Mode
      in this process. (As you see in Downlink Framestructure,
      MIB/PBCH is located at the 6 RBs around the center frequency.
      So the success of MIB decoding does not guarantee that signal
      quality across the whole band is good)
* Detect CSR (Cell Specific Reference Signal) and perform Channel
  Estimation and Equalization. In this process, UE will detect/measure
  reference signal across the whole system bandwidth. So RSRP/RSRQ
  measured at this step can be a good indicator for overall signal quality.
* Decode PDCCH and extract DCI information for SIB. PDCCH is spread
  across the whole bandwidth, so the signal quality across the whole
  bandwidth should be good enough for this step.
* SIB deconding (SIB1 should be decoded first and then SIB2 and then
  remaining SIBs)
* Cell Selection:
    * UE may find multiple suitable cells, but it try camp on to HPLM
      cell with the highest priority

## DRX

* A way for the network to let the UE sleep for some time and only wake up for the moments
  when the network will schedule data for that UE.

# Procedures

* Create Session Request/Response
    * between MME and SGW for creating a session
* Modify Bearer Request/Response
    * between MME and SGW for updating a session

# Radio stuff

1 s  =  100 Frames
     =  10 sub-frames per frame * 100    =  1000 sub-frames
     =  2 slots per sub-frame * 1000     =  2000 slots      (each slot = 0.5 ms)

There are 7 symbols per slot, with each symbol preceded by a cyclic-prefix.
Or there are 6 symbols per slot, with each symbol preceded by a extended-cyclic-prefix

You have a Band, Say band-48 (CBPRS), this is split into Bandwidth of either of the following.
This Bandwidth is called the carrier and is the least unit given to one cell.
But note that with a re-use factor of 1, adjacent cells can have the same freq.

Bandwidth | Resource Bloc | Subcarriers (downlink) | Subcarriers (uplink)
1.4 MHz   | 6             | 73                     | 72
3 MHz     | 15            | 181                    | 180
5 MHz     | 25            | 301                    | 300
10 MHz    | 50            | 601                    | 600
15 MHz    | 75            | 901                    | 900
20 MHz    | 100           | 1201                   | 1200

One subcarrier is 15Khz.
One Resource Element is 1 subcarrier * 1 Symbol
One Resource Block is 12 subcarriers (180Khz) * 1 time-slot (7 symbols)

In a 1.4Mhz bandwidth allocation, there are 6 Resource blocks

How you allocate the 10 subframes is the TDD config.

# CBRS stuff

* Different frame structure can cause bad interface.
    * One is doing downlink(higher power) when another guy is doing uplink(lesser power)
* CA is usually not sported in uplink, as band-48 is mostly used as downlink offload by carriers
* Band-48 is 150Mhz big. Its typically split into 20/10 sized channels. (15/5 is allowed, but rarely used in practise)
* Types of categories
    * Cat-A lowpower
        * `< 200mW`
        * phones typically transmit at this power
    * Cab-B high power
        * 47dbMs, 30W-per-10Mhz
        * 6M above ground
        * can power 2x2 sq.miles
* direction of antennas
    * omni, direction, tilted
    * back lobe, back leakage
* cat-B cpe should be static
    * rural internet is a typical use-case.
* CPRI - common public radio interface
    * interface between RRU (the antenna unit) and the baseband unit (central processing unit).
    * This is typically about 10 miles long
    * eCPRI - enhanced CPRI.
    * stretches the limits of fiber bw with 5G rates.

# from tutorials

Dump:

```
Evolved Packet System (EPS) bearers provide one-to-one correspondence with RLC radio bearers and provide support for Traffic Flow Templates (TFT). There are four types of EPS bearers:
* GBR Bearer resources permanently allocated by admission control
* Non-GBR Bearer no admission control
* Dedicated Bearer associated with specific TFT (GBR or non-GBR)
* Default Bearer Non GBR, catch-all for unassigned traffic


A home eNB (HeNB) is a base station that has been purchased by a user to provide femtocell coverage within the home. A home eNB belongs to a closed subscriber group (CSG) and can only be accessed by mobiles with a USIM that also belongs to the closed subscriber group.

```

* https://www.tutorialspoint.com/lte/index.htm
* https://go.radisys.com/rs/radisys/images/paper-lte-protocol-signaling.pdf


