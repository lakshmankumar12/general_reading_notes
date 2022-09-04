# General Idea

* You will see lots of Network Function (NF)
* Service based architechture
* Service points (Rest/API)
* Point-to-Point (like in 4G)

## 3 use-cases 5G tries to solve

* eMBB  -- enhanced Mobile Broadband (better 4G network)
* URLLC -- Ultra Reliable Low Latency Connection
* mMTC  -- massive Machine Type Communication

# Spec List

* Refer to sae_lte_understanding.md

# List of Point to Point interfaces

N1  -   UE to AMF
N2  -   NR to AMF
N3  -   NR to UPF
N4  -   SMF to UPF
N6  -   UPF to Data-Network
N9  -   UPF to UPF
N11 -   AMF to SMF (Mnemonic: S11 in 4g is MME -- SGW)



# States

* Registraion Managment(RM)
    * RM - Registered
    * RM - DeRegistered
* Connection Management
    * IDLE
        * RRC-IDLE
    * Connected
        * In RRC-Active or RRC-Inactive
* RRC states
    * NR-RRC-Idle
    * NR-RRC-Connected
    * NR-RRC-Inactive
        * NR-locally manages paging
        * Helps to quickly come back to Connected w/o re-negotiation auth/security
* Service and Session Continuity Modes (SSC modes)
    * How IP continuity is maitained when the PDU-Session-Anchor(UPF) changes.
    * SSC Mode-1
        * Standard - Keep the same PSA no matter which NR is the current one.
    * SSC Mode-2
        * PSA changes. So IP of the UE gets Changed.
        * Network releases the pdu and indicates the UE to start establishing
          data-session with the same DNN/S-NSSAI to regain PDU continuity
        * break-before-make
    * SSC Mode-3
        * UE maintains connectivity with both PSA(s)!, so its multi-homed with both IPs
        * make-before-break
        * Eventually the old session is closed.

# Simplified Diagram

```

  UE ----- NR --+----  AMF  ----  SMF
                |
                |
                +---- UPF ---- Data-Network


```

# Core network Elements

NR    -  New Radio (GNodeB)
AMF   -  Access and Mobility Function
SMF   -  Session Management Function
LMF   -  Location Management Function
PCF   -  Policy Control Function
AUSF  -  Authentication and Subscription Function
UDM   -  Unified Data Management
UDR   -  Unified Data Repository
UPF   -  User plane function
CHF   -  Charging control function
NRF   -  Network Repository function
NEF   -  Network Exposure Function
5GEIR -  5G Equipment Identity Registry
NSSF  -  Network Slice Selection Function

## Details

* AMF
    * Very similar to MME, but there are some differences too.
    * Relays messages to UE from all of SMF, AUSF, UDM, SMSF, PCF
    * One UE is connected to only one AMF at any point in time.
    * Responsible for picking SMF (like MME chose SGW/PGW in 4G)
* SMF
    * Responsible for setting up User-plane for the UE.
    * Allocates IP to the UE.
    * Alloctes, modifies and releases session.
    * A bit similar to a collapsed SGW/PGW(its control function) in 4G.
    * It actually anchors the bearer related messages with the UE (in this way,it
      takes some MME work on it as well)
    * It interacts with PCF for policies
    * It interacts with CHF, it collects the charging records for the user.
    * Downlink Data Notification(DDN) is sent by SMF to AMF
* UPF
    * Handles data from UE
    * Generates charging data records and user data records and sends to SMF
    * BUffering of Downlink data (till paging is complete)
    * Serves as the PSA (PDU Session Anchor) , the IP-Anchor for the UE.
    * Applies QoS in the downlink direction
    * UPFs can be deployed in series (unlike in 4G!), with each UPF doing some job
        * Usecases:
            * Network-wide mobility
                * one UPF acts as PSA, and the current UPF forwards to this UPF
            * Breakout of select data flows
                * SMF can implement ULCL (Uplink Classifiers) for a given user and
                  route data flows differently based on filters(eg: 5-tuple).
                * Helps to implement MEC (Mobile Edge computing)
            * Roaming with Home routing
* NRF
    * Enables service discovery.
        * Avoids static mapping of services to their endpoint addresses
    * Services register with NRF at their start telling about themselves
    * When services need to contact other services, they query NRF to get details of their
      target service
* UDM
    * Frontend for UDR, that stores all subscription information
    * Uses subscription data to perform
        * access authorization
        * registration management
        * reacheability for terminating events like SMS
    * Handles the SUCI to SUPI convertion
    * Knows which AMF is handling the user
    * Always in the home plmn
    * Similar to HSS in some aspects
* UDR
    * Stores subscription like
        * data related to network
        * policies
    * Helps UDM, PCF, AUSF and NEF
* AUSF
    * In the home network
    * Perform auth based on info from UE
* PCF
    * provides policy control for
        * access and mobilty mgmt
            * frequency selection priority (used by RAN)
        * session mgmt
            * QoS authorized
            * Charging control
            * Policy Control
            * Event reporting
    * can directly interact with UE via AMF for
        * discovery/selection of non-3GPP networks
        * session continuation mode selection
        * network slice selection
        * data name selection
* NSSF
    * AMF asks the NSSF to give the list of AMF(s) that can serve a give UE
      for its requested Network-slice.
    * There is a NAS Reroute message, which a AMF can send to gNB to make it
      send the re-route message to the other AMF
    * Alternatively AMF can relay the registration with another AMF as well.

# QoS

* 4G calls a session as PDN-connection, while 5G calls it PDU-session

* Qos Flows
    * Finest granulatiry of QoS Differentiation
    * Has a Qos Flow ID
    * Multiple Qos-Flows may map to a radio bearer (in contrast with 4g, where one QCI==1 Bearer)
        * Gnodeb and the N3-tunnel are aware of the Qos-Flow-Id within the bearer.
* Qos Profile
    * Identified by 5G-Qos Identifier -- 5QI and ARP.
* PDR
    * Packet detection Rules.
    * Installed by SMF into the UPF, which in turn are obtained from PCF
    * Classifies pkts
    * Maps, say a 5-tuple to a Qos-Flow-Id
    * Installed both at UPF (for downlink) and at UE (for uplink)
* Reflective Qos
    * Requires support for UE
    * Helps to reduce signalling. It basically tells the UE to do the same QoS treatment to the
      uplink packet for the downlink packet received.
      * Install PDRs only on UDF, and use refletive Qos to install PDRs in UE
    * The downlink packet in its header has a way to tell the UE to note that that Reflective Qos
      is enabled on this pkt (yes, per pkt basis) and install the PDRs appropriately for uplink.
      * This is referred as RQI marking on the pkt.
    * There is a reflective Qos timer to guard how long the PDRs remain in action.

# Identities

* SUCI
    * Subscriber concealed identity
    * One time usage. Everytime its used, a new SUCI is used.
    * Is produced with the Home-Network-Public-Key
    * In IMSI based SUPI, the MSIN is used to compute.
* SUPI
    * Subscriber Private identity , like IMSI(need not be always IMSI)
    * Globally unique, provisioned in UDR
    * Shall contain the address of home network
    * For interworking with EPC(4G), SUPI shall be IMSI-based.
    * Never transmitted on air.
* PEI
    * IMEI or IMEISV
    * Same as EPS
* 5G-GUTI
    * Globally unique temporary identifier (GUTI)
    * Globally unique AMF Identifier (GUAMI)
    * Assigned as part of AMF, as part of Registration-Accept
    * Components
      ```
      5G-GUTI =   5G-GUAMI                               +   M-TMSI
              = MCC + MNC + AMF      + AMF   +  AMF      +   M-IMSI
                            RegionId   SetID    Pointer
                            8 bits     10 bits   6 bits
      ```
    * There is a scheme to convert 4G-GUTI <---> 5G-GUTI. Actually quite straightforward, as
      the number of bits in 4G-GUTI and 5G-GUTI is still the same

      ```
           4G                 5G
           MCC            ==  MCC
           MNC            ==  MNC
           MMEGI (8 bits) ==  AMF-Region-ID (8 bits)
           MMEGI (8 bits)
           +              ==  AMF-Set-ID (10 bits)
           MMEC  (2 bits)
           MMEC  (6 bits) ==  AMF-Pointer (6 bits)
           M-TMSI         ==  5G-TMSI
      ```
      In co-located AMF/MME, the above mapping makes it seamless.

# Network Slicing

* Multiple logical network tailored for different use cases like eMBB, URLLC and mMTC
* Slice Type(ST) - 8 bits, Slice Differentiator(SD) - 24 bits
* S-NSSAI - Single Network Slice Selection Assistance Information
* Can be use-case specific or UE-specific


# Security

* Network Access Security
    * Mobile connection with the Air Network
* Network Domain Security
    * Inter NF Security (eg: RAN to AMF, or AMF to UDM)
* User Domain Security
    * ME and USIM
* Application Domain sercurity
    * App in UE to the Services in Data-Network
* SBA domain security
    * Between Operators
