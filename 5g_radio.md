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
    * (CP,useful) symbols
    * First OFDM symbol is slighly longer than the rest
* So, one slot has 2048*7 = 14336 useful samples, 1204 CP samples,
  15360 samples in all.
* This 2048 useflu symbols is referred as 2048 bins/IFFT (N_ifft)


* Resource Grid - time is X-axis , Frequency in Y-axis
* Resource Element - most minimal - 1 sub-carrier x 1 symbol
* Resource Block -- Mostly discussed.
    * 7 symbols(1 slot) x 12 subcarrier

* In a 20Mhz system bandwith, there will be 100 RBs - 1200 subcarriers
* In a 10Mhz system bandwith, there will be  50 RBs -  600 subcarriers
* In a  5Mhz system bandwith, there will be  25 RBs -  300 subcarriers

* subcarriers are of types
    * Guard -> that last 2 on either side
    * Pilot -> ..
    * Data  -> real symbols carrying
    * DC    -> center subcarrier w/o data

TDD configs

* splits how the 10 subframes in a frame is split for uplink/downlink/special-subframe

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


# Antenna ports

https://www.sharetechnote.com/html/Handbook_LTE_AntennaPort.html




# Terms

* Frequence Range (FR) - FR1 is `< 6Ghz` while FR2 is `24 to 52 Ghz` (millimeter)
* Cyclic Prefix (CP)
* Protocol Resource Block
* max BW is 100Mhz for sub6Ghz, and 400Mhz for mm-wave
    * 5/10/.../100 for sub6
    * 50/100/200/400 for mm
* Subcarrier spacing/size (μ) - 15Khz, 30Khz, 60, 120, 240, 240, 480
    * 60 and above is for mm-wave
* CA allows upto 16 carriers
* 256 QAM
* MIMO - upto 8 layers

# Frames

* 1 Frame is 10ms long
* 1 Frame has N, (μ)-sized sub-frames
*


# A bit radio'ey


* Synchronization configurations
    * LLC-C1, .. LLS-C4
