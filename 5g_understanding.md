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

* Important ones - N3, N4, N11

N1  -   UE to AMF
N2  -   NR to AMF
N3  -   NR to UPF
N4  -   SMF to UPF
N6  -   UPF to Data-Network
N7  -   SMF to PCF
N8  -   AMF to UDM
N9  -   UPF to UPF
N10 -   SMF to UDM
N11 -   AMF to SMF (Mnemonic: S11 in 4g is MME -- SGW)
N15 -   AMF to PCF
N26 -   MME to AMF





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
UDM   -  Unified Data Management
UDR   -  Unified Data Repository
UPF   -  User plane function
CHF   -  Charging control function
NRF   -  Network Repository function
NEF   -  Network Exposure Function
5GEIR -  5G Equipment Identity Registry
NSSF  -  Network Slice Selection Function
AUSF  -  Authentication Server Function
SEAP  -  Security Anchor Function
ARPF  -  Authentication Credential Repository and Processing Function
SIDF  -  Subscriber Identity De-concealing Function
AF    -  (External) Application Function
NWDAF -  ?? (inter-working something)


## Details

* AMF
    * Very similar to MME, but there are some differences too.
    * Relays messages to UE from all of SMF, AUSF, UDM, SMSF, PCF
    * One UE is connected to only one AMF at any point in time.
    * Responsible for picking SMF (like MME chose SGW/PGW in 4G)
    * Implements the SEAP function to enforce authentication
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
* AUSF
    * Responsible for authentication, given data from UE and uses services of UDM
    * Always in HPLMN
* SIDF
    * Converts SUPI to SUCI.
    * Always in HPLMN
    * Can be within UDM or UDR (impl. dependant)
* ARPF
    * Stores the credentials - Keys and SUPI
    * Can be within UDM or UDR (impl. dependant)
* NEF
    * Exposes 5G data to other networks like UE location/mobility-status etc.
* AF
    * External application that can create policies into the 5G-CN(PCF) for UEs

## services offered

* AMF
    * Table 5.2.2.1-1 in 23.502
    * Namf_Communication
        * allows other services like SMF,PCF to communicate to UE over NG-RAN
        * enables one AMF to fetch contexts from another AMF
        * allows for subscription(by other services) to UE status changes
    * Namf_EventExposure
        * allows other services to subscribe to events such as UE recheability, mobility change
    * Namf_MT
        * allows other services to ensure UE is recheable
    * Namf_Location
        * allows other service to get location of UE
* SMF
    * Table 5.2.8.1-1 in 23.502
    * Nsmf_PDUSession_service
    * Nsmf_EventExposure
* PCF
    * Npcf_AMPolicyControl
        * access-control, network-selection, mobility-mgmt-related policies, ue-route-selection
        * allows AMF to create/modify/delete per UE policy assocations with PCF
        * Per-UE policies can have access and mobility policy info, policy-control-request trigger conditions
            * Upon hitting trigger AMF may contact PCF, which may give further policies to AMF
            * allows PCF to send new Policies for a existing AMF-PCF per UE policy association
    * Npcf_PolicyAuthorization
        * Authorizes and crates policies from AF(s) for the pdu-session for which the AF is bound to
        * allows SMF to create/modify/delete per UE policy assocations with PCF
        * same as AMF - send in triggers and send more policies when triggers are met.
    * Npcf_SMPolicyControl
        * PDU session related policies for SMF
    * Npcf_BDTPolicyControl
        * Background Data Transfer (BDT) policies for NEF
    * Npcf_UEPolicyControl
    * Npcf_EventExposure
        * allows other service to subscribe to events (eg, IPchange, Data-usage-cap-reached)
* UDM
    * Table 5.2.3:1-1 in 23.502
    * Nudm_SDM - subscriber data mgmt
        * AMF, SMF get subscription data from here.
        * And notified of new subscription updates
    * Nudm_UECM - UE context mgmt
        * AMF, SMF can update the UDM of the current AMF,SMF serving the UE, UE-status etc.
    * Nudm_UEAuthentication
    * Nudm_EventExposure
    * Nudm_ParameterProvision
    * Nudm_NIDDAuthorization
* NRF
    * Nnrf_NFManagement
    * Nnrf_NFDiscovery
    * Nnrf_AccessToken

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
* DNN
    * Data network Name
    * provided by UE when initiating a pdu-session-request
* PDU session id
    * Identifies the pdu-session. (created and owned by the SMF)

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

## other points

* There is integrity protection for data, which isn't there in 4g.
  4G has only ciphering.
* Home plmn is more involved in authentication process
* UDM decides integrity and ciphering protection instad of enodeb.
* AKA - sim based auth
* EAP-AKA also possible.
* See figure 6.2.1-1 in TS.33.501 for the key hierarchy
    * In short, this is the hiearchy of functions and keys
        ```
         UDM/ARPF -> UDM/ARPF -> AUSF -> SEAF -> AMF -> Gnb
         K -> CK,IK -> KAUSF -> KSEAF -> KAMF --+--> K-Nas-int, K-Nas-enc
                                                |
                                                +--> Kgnb --+--> K-rrc-int
                                                            |
                                                            +--> K-rrc-enc
                                                            |
                                                            +--> K-up-int (new in 5g)
                                                            |
                                                            +--> K-up-enc
        ```
    * The K-AUSF has the serving plmn as its input. This ensures, different
      keys are used in different networks.

* Authentication vector
    * Has 5 components
        * AUTN - authentication token
        * RAND
        * XRES* - AUSF computes HXRES* and stores XRES*
        * K_AUSF
    * AUSF forwards only (AUTN, RAND, HXRES*) to the serving network
    * K + AUTN ---> CK, IK, RES (given by USIM)
    * CK, IK, RES ---> RES*, K_AUSF, K_SEAF, KAMF (done by ME)

# Procedures

## Registration

* 23.502 has it
* first procedure after power on
* types:
    * initial reg, mobility reg, periodic reg
* Info sent by UE:
    * IDentities - 5G-TMSI/GAUMI or SUCI
    * registration type
    * requested-NSSAI
        * Help RAN pick the AMF that serves this requested-NSSAI.
        * If RAN can't pick, it will send to default AMF
* New AMF contacts old AMF to collect UE-context

