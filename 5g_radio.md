# links

* https://www.rfwireless-world.com/Tutorials/5G-tutorial.html
* Splits in 5G Radio architechture
    * https://www.5gtechnologyworld.com/open-ran-functional-splits-explained/
* https://www.rfwireless-world.com/Articles/5G-NR-Physical-Layer.html

# LTE Frame structure

https://www.sharetechnote.com/html/FrameStructure_DL.html

* 1 Frame - 10ms
* 1 subframe - 1ms (10 subframes in 1 frame)
* 1 slot - 0.5ms (2 slots in 1 subframe)
* 1 slot -- 7 OFDM symbol blocks
    * One symbol is a certain time span of signal that carry one
      spot in the I/Q constellation.
* Beginning of symbol is a cyclic prefix
    * If we use extended cyclic prefix (longer duration), we can
      have only 6 symbols
* The first CP is slightly longer. So,
    * first cp time    -  5.7 us
    * 2-7   cp time    -  4.2 us
    * All symbols time - 66.7 us
    * Total = (66.7+4.7)*6 + 66.7 + 5.2 = 500.3 (ms)

At 30.72 Mhz sampling rate:
* One frame can carry 307,200 samples (307.2K)
* One slot is 15,360 samples (307.2K/20)
* One slot has (160+2048) in first and (144+2048) in next 6.
    * (CP,useful) samples
    * First OFDM symbol is slighly longer than the rest
* So, one slot has 2048*7 = 14336 useful samples, 1204 CP samples,
  15360 samples in all.
* This 2048 useful samples in one symbol, is referred as 2048 bins/IFFT (N_ifft)


* Resource Grid - time is X-axis , Frequency in Y-axis
* Resource Element - most minimal - 1 sub-carrier x 1 symbol
* Resource Block -- Mostly discussed.
    * 7 symbols(1 slot) x 12 subcarrier

* In a 20Mhz system bandwith, there will be 100 RBs - 1200 subcarriers
* In a 10Mhz system bandwith, there will be  50 RBs -  600 subcarriers
* In a  5Mhz system bandwith, there will be  25 RBs -  300 subcarriers

* subcarriers are of types (to confirm - may be in LTE all are data only)
    * Guard -> that last 2 on either side
    * Pilot -> ..
    * Data  -> real symbols carrying
    * DC    -> center subcarrier w/o data

TDD configs

* splits how the 10 subframes in a frame is split for uplink/downlink/special-subframe

# basic procedure overview

https://www.sharetechnote.com/html/BasicProcedures_LTE.html

# Channel types

* PBCH (Physical Broadcast Channel)
    * carries MIB(Master Information Block)
    * 6 resources blocks around centered around DC on first sub-frame
* PCFICH(Physical Control Format Indicator Channel)
    * carries number of symbols use for control channels (PDCCH and PHICH)
* PDCCH(Physical Downlink Control Channel)
    * carries DCI (Downlink Control Indicator)
    * Multiple PDCCH can be assigned in single subframe and a UE do blind
      decoding of all the PDCCHs.
* PHICH
    * Carries H-ARQ Feedback for the received PUSCH
    * After UE trasmitted the data in UL, it is waiting for PHICH for the ACK.
* PDSCH(Physical Downlink Shared Channel)
    * Carries user specific data (DL Payload).
    * Carries Random Access Response Message.
* PRACH(Physicall Random access channel)
    * uplink channel
    * carries random access preamble
* P-SS(Primary Synchronization Signal)
    * Mapped to 72 active sub carriers(6 resource blocks), centered around
      the DC subcarrier in slot 0 (Subframe 0) and slot 10 (Subframe 5).
    * Made up of 62 Zadoff Chu Sequence Values
    * Used for Downlink Frame Synchronization
    * One of the critical factors determining Physical Cell ID
* S-SS(Secondary Synchronization Signal)
    * Mapped to 72 active sub carriers(6 resource blocks), centered around
      the DC subcarrier in slot 0 (Subframe 0) and slot 10 (Subframe 5) in FDD.
    * The sequences are different in 0 and 5.
    * Used for Downlink Frame Synchronization
    * One of the critical factors determining Physical Cell ID
* RS (Reference Signal ) - Cell Specific
    * Exists only at PHY layer
    * Reference point for downlink power
    * Cell Specific Reference Signal
        * This reference signal is being transmitted at every subframe and
          it spans all across the operating bandwidht. It is being transmitted
          by Antenna port 0,1,2,3.
        * UE Specific Reference Signal : This reference signal is being
          transmitted within the resource blocks allocated only to a specific
          UE and is being transmitted by Antenna port 5.
* PUSCH
    * uplink channel
* PUCCH
    * uplink channel

# Antenna ports

https://www.sharetechnote.com/html/Handbook_LTE_AntennaPort.html


# 5G specific

* Frequence Range (FR) - FR1 is `< 6Ghz` while FR2 is `24 to 52 Ghz` (millimeter)
* max BW is 100Mhz for sub6Ghz, and 400Mhz for mm-wave
    * 5/10/.../100 for sub6
    * 50/100/200/400 for mm
* Subcarrier spacing/size (μ) - 15Khz, 30Khz, 60, 120, 240, 240, 480
    * 60 and above is for mm-wave
* CA allows upto 16 carriers
* 256 QAM
* MIMO - upto 8 layers

## 5G frame splits

Subcarrier spacing (KHz)     | 15   | 30   | 60                                  | 120  | 240
Symbol duration (µs)         | 66.7 | 33.3 | 16.7                                | 8.33 | 4.17
CP duration (µS)             | 4.7  | 2.3  | 1.2 (Normal CP), 4.13 (Extended CP) | 0.59 | 0.29
Max. nominal system BW (MHz) | 50   | 100  | 100 (sub-6 GHz), 200 (mmwave)       | 400  | 400
FFT size (max.)              | 4096 | 4096 | 4096                                | 4096 | 4096
Symbols per slot             | 14   | 14   | 14 (normal CP), 12 (extended CP)    | 14   | 14
Slots per subframe (Note1)   | 1    | 2    | 4                                   | 8    | 16
Slots per frame              | 10   | 20   | 40                                  | 80   | 160
Subframes per frame (fixed)  | 10   | 10   | 10                                  | 10   | 10

Note1 - Each subframe has `2^μ` slots, depending on subcarrier size.
Note2 - 4G had 7 or 6(ext-CP) symbols per slot. Here its 14 (or 12).

* 12 subcarriers in one Resource-Block
* 4G had eactly 20 RBs in one frame (20 slots in one frame), and 12 subcarrier in freq-axis
  In a 20Mhz BW, there would be hence 100RBs (1200 subcarriers)
  * 5G NR supports 24 to 275 PRBs in a single slot (in the freq-axis)


# functional split

* RU:
    * This is the radio hardware unit that coverts radio signals sent to and
      from the antenna into a digital signal for transmission over packet
      networks.
    * It handles the digital front end (DFE) and the lower PHY layer, as well
      as the digital beamforming functionality.
    * key considerations of RU design are size, weight, and power consumption.
    * Deployed on site.
* DU (Distributed Unit)
    * Deployed on site on a COTS server.
    * Normally deployed close to the RU on site
    * Runs the RLC, MAC, and parts of the PHY layer.
    * Includes a subset of the eNodeB (eNB)/gNodeB (gNB) functions, depending on
      the functional split option, and its operation is controlled by the CU.
* CU (Centralized Unit)
    * Runs the Radio Resource Control (RRC) and Packet Data Convergence Protocol (PDCP) layers.
    * gNB consists of a CU and one DU connected to the CU via Fs-C and Fs-U interfaces
      for CP and UP respectively.
    * A CU with multiple DUs will support multiple gNBs. The CU controls the operation of
      several DUs over the midhaul interface.
        * CU software can be co-located with DU software on the same server on site.
    * The split architecture lets a 5G network utilize different distributions of protocol
      stacks between CU and DUs depending on midhaul availability and network design.
    * Includes the gNB functions like transfer of user data, mobility control,
      RAN sharing (MORAN), positioning, session management etc., except for functions
      that are allocated exclusively to the DU.

* ECPRI -> interface between DU and RU, called fronthaul
    * fronthaul latency should be less than 100us.
    * Fronthaul - CUS plane (Control/User/Sync) and M plane (mgmt plane)
* F1 -> midhaul between CU and DU
    * latency should be less than 1ms

# layers

* RF , PHY, MAC, RLC, PDCP, RRC/DATA
* PHY/MAC/RLC have LO/HI
* RF/PHY are L1, MAC/RLC are L2, PDCP/RRC are L3
* 7.2x split is keeping RF,LOW-PHY in RU
* typically HI-PHY/LOW&HI-MAC/LOW&HI-RLC are with DU. Rest in CU


```
control plane
-------------

UE        RU               DU                  CU        AMF/Core

NAS-msg                                                  NAS-msg
RRC                                     RRC
PDCP                                    PDCP
RLC                   RLC      F1-AP    F1-AP  NG-AP     NG-AP
MAC                   MAC      SCTP     SCTP   SCTP      SCTP
PHY       LOW-PHY     HI-PHY   IP       IP     IP        IP

Data plane
----------

UE       RU          DU                  CU                  UPF/Core


IP...........................................................IP
SDAP                                   SDAP(PSUP)
PDCP                                   PDCP(NRUP)
RLC                  RLC      eGTPU    eGTPU       eGTPU     eGTPU
MAC                  MAC      UDP      UDP         UDP       UDP
PHY      LOW-PHY     HI-PHY   IP       IP          IP        IP

```

# mac layer

* HARQ - hybrid automatic repeat request
* BSR  - buffer status report (from ue to network)
* DRX  - discontinuous reception mode 
         (enables ue to relax and check signals once in
          a while. Great battery saver)
* CQI  - Channel Quality Indicator measurements

* Maps logical to physical/transport channels
* SDUs belongs to logical channels and transport blocks belong to physical
    * So, this multiplexes and de-multiplexes this.

## logical channels

* BCCH - broadcast control channel
* PCCH - paging    control channel
* CCCH - common    control channel
* DCCH - dedicated control channel
* DTCH - dedicated traffic channel

Uplink Maps
Physical ->  PUSCH     PRACH
Logical
CCCH         X
DCCH         X
DTCH         X

Downlink Maps
Physical ->  PBCH      PDCCH   PDSCH
Logical
BCCH         X                   X
PCCH                    X
CCCH                             X
DCCH                             X
DTCH                             X

## Procedures

* Random Access Procedure
    * Get the initial uplink grant for UE and helps in performing
      synchronization with the gNB (i.e. network).
    * It covers Random Access procedure
        * procedure initialization,
        * Resource selection,
        * Preamble transmission,
        * Response reception,
        * Contention Resolution
        * Completion
* DL-SCH transfer
* UL-SCH transfer
* Scheduling Request
    * Used by UE to transmit request to gNB (i.e. network) to obtain UL grant.
* PCH reception
    * monitoring paging in special time period
* BCH reception
    * MIB, SFN etc.
* DRX
    * monitoring PDCCH as per special pattern in discontinuous manner.
* 




# Processing

* Upper layer gives transport blocks

```
    TransportBlock
          |
          |
          v
    CRC Attachment
          |
          |
          v
    LDPC base graph selection
    Code block segmentation and CRC attachment
    LDPC encoding
    Rate matching
    Code block concatenation
    Scrambling
    Modulation
    Layer Mapping
          |
          |(multiple)
          v
    Antenna Port mapping
    Mapping to RBs

```

docs:
3GPP TS 38.201 : General description
3GPP TS 38.202 : Services provided by physical layer
3GPP TS 38.211 : Physical channels and modulation
3GPP TS 38.212 : Multiplexing and channel coding
3GPP TS 38.213 : Physical layer procedures for control
3GPP TS 38.214 : Physical layer procedures for data
3GPP TS 38.215 : Physical layer measurements

3GPP TS 38.374 : IAB mentions

# IAB

* Integrated access and backhaul
* CU is single. THe DU keeps getting split as donor-DU and IAB-DU.
* IAB has 2 parts:
    * IAB joins the network as a regular UE all the way to the core
      using its MT(Mobile-termination)
        * Similar to gxc solution in 4G.
    * Then IAB-DU setup happens
* Note that CU is central. Only DU functionality is split.
* BAP - backhaul adaptation protocol
    * On top of RLC. So, PHY,MAC,RLC repeat. But F1-AP is UE to CU.
* The interface between IAB to its donor is F*
* RRC Connection establishment and F1-setup request indicates IAB support

```
control plane
-------------

UE        RU       Node1-DU   Node2-DU      DU        CU             AMF/Core

NAS-msg                                                                NAS-msg
RRC                                                   RRC
                    F1-AP                             F1-AP
                    SCTP                              SCTP
                    IP        IP  IP        IP        IP
PDCP                BAP       BAP BAP       BAP       PDCP
RLC                 RLC       RLC RLC       RLC       F1-AP  NG-AP     NG-AP
MAC                 MAC       MAC RLC       RLC       SCTP   SCTP      SCTP
PHY       LOW-PHY   PHY       PHY RLC       RLC       IP     IP        IP

Data plane
----------

UE        RU       Node1-DU   Node2-DU      DU        CU             AMF/Core

UE-IP                                                                  UE-IP
SDAP                                                  SDAP
PDCP                                                  PDCP
                    GTPU                              GTPU
                    UDP                               UDP
                    IP        IP  IP        IP        IP
                    BAP       BAP BAP       BAP
RLC                 RLC       RLC RLC       RLC              GTPU      GTPU
MAC                 MAC       MAC RLC       RLC              UDP       UDP
PHY       LOW-PHY   PHY       PHY RLC       RLC              IP        IP


```

# Antenna related terms

* MIMO
* Spatial diversity
* Sectors
* Ports

# Other Dump of terms


* Synchronization configurations
    * LLC-C1, .. LLS-C4
