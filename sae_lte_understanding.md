SAE and LTE General Understanding
==================================
:toc:

# List of interfaces

* S1   -  Data plane from Enb to SGW
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
* S8   -  inter-plmn S5->S9
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
* SGi

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
23.502


# IDs

* PLMN-ID   = MCC + MNC
* MMEI      = MMEC + MMEGI
    * MME-code (one mme withing the group)
    * MME-group-identifier
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

* The PGW and PCRF maintain session-level contexts.
* Each session is identified at PGW by (IMSI,APN-requested,Access-type) and so is it as PCRF
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
  the rules that are applied/in-progress
  for that session.
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


