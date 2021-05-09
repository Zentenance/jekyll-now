---
title: "PCIe Configuration space in Windows driver’s point of view"
date: 2021-05-08
draft: false
tags: ['Embedded Software', 'Windows', 'Device Driver']
---

## What is PCIe?
PCI Express, technically Peripheral Component Interconnect Express is a standard type of connection for internal devices in a computer. 

1. Synchronous 
2. Transaction / Burst Orientated 
3. Bus Mastering 
4. Plug and Play

## Debug PCIe using WinDbg
### PCI Type 0 Configuration Space Header
 
The _!pci_ extension displays the current status of the peripheral component interconnect (PCI) buses, as well as any devices attached to those buses.

```
!pci [Flags [Segment] [Bus [Device [Function [MinAddress MaxAddress]]]]]
```

For example, when flags is 301, it means; 
* 0x200:Causes the display to include segment information. When this bit is included, the _Segment_ parameter must be included
* 0x100: Causes the display to include the PCI configuration space 
* 0x001: Causes a verbose display. 

The picture below is the standard registers of PCI Type 0 (Non-Bridge) Configuration Space Header. 
![](/images/PCIe_Configuration_space_in_Windows_drivers_point_of_view/F43FB314-5ADA-49A2-98F9-6DE54D5289E7.png)

Here is the example;

```
    0: kd> !pci 301 0 1 0 0
    PCI Configuration Space (Segment:0000 Bus:01 Device:00 Function:00)
    Common Header:
        00: VendorID       XXXX 
        02: DeviceID       XXXX
        04: Command        0406 MemSpaceEn BusInitiate InterruptDis 
        06: Status         0010 CapList 
        08: RevisionID     02
        09: ProgIF         00
        0a: SubClass       80 Other Network Controller
        0b: BaseClass      02 Network Controller
        0c: CacheLineSize  0000
        0d: LatencyTimer   00
        0e: HeaderType     00
        0f: BIST           00
        10: BAR0           60c00004
        14: BAR1           00000000
        18: BAR2           00000000
        1c: BAR3           00000000
        20: BAR4           00000000
        24: BAR5           00000000
        28: CBCISPtr       00000000
        2c: SubSysVenID    1ae9
        2e: SubSysID       0000
        30: ROMBAR         00000000
        34: CapPtr         40
        3c: IntLine        00
        3d: IntPin         01
        3e: MinGnt         00
        3f: MaxLat         00
    Device Private:
        40: 0003b001 00000008 00000000 00000000
        50: 00000000 00000000 00000000 00000000
        60: 00000000 00000000 00000000 00000000
        70: 00020010 10008fc0 00102010 00434812
        80: 00120042 00000000 00000000 00000000
        90: 00000000 0000081f 0000000a 00000006
        a0: 00000002 00000000 00000000 00000000
        b0: 00a57005 17a10040 00000000 000002a4
        c0: 00000000 00000000 00000000 00000000
        d0: 00000000 00000000 00000000 00000000
        e0: 00000000 00000000 00000000 00000000
        f0: 00000000 00000000 00000000 00000000
    Capabilities:
        40: CapID          01 PwrMgmt Capability
        41: NextPtr        b0
        42: PwrMgmtCap     0003 Version=3
        44: PwrMgmtCtrl    0008 DataScale:0 DataSel:0 D0 
        b0: CapID          05 MSI Capability
        b1: NextPtr        70
        b2: MsgCtrl        64BitCapable MSIEnable MultipleMsgEnable:2 (0x4) MultipleMsgCapable:2 (0x4)
        b4: MsgAddr        17a10040
        b8: MsgAddrHi      0
        bc: MsData         2a4
        70: CapID          10 PCI Express Capability
        71: NextPtr        00
        72: Express Caps   0002 (ver. 2) Type:Endpoint
        74: Device Caps    10008fc0
        78: Device Control 2010 bcre/flr MRR:512 ns ap pf et MP:128 RO ur fe nf ce
        7a: Device Status  0010 tp AP ur fe nf ce
        7c: Link Caps      00434812
        80: Link Control   0042 es CC rl ld RCB:64 ASPM:L1 
        82: Link Status    0012 scc lt lte NLW:x1 LS:2.5 
        94: DeviceCaps2    0000081f CTR:15 CTDIS arifwd aor aoc32 aoc64 cas128 noro LTR TPH:0 OBFF:0 extfmt eetlp EETLPMax:0
        98: DeviceControl2 000a CTVal:10 ctdis arifwd aor aoeb idoreq idocom ltr OBFF:0 eetlp
        84: Slot Caps      00000000 
        88: Slot Control   0000 pcc PI:?? AI:?? hpi cc pde mrls pfd ab 
        8a: Slot Status    0000 dlsc eis pds hpi cc pdc ms pfd ab 
        8c: Root Control   0000 pmei fs nfs cs 
        8e: Reserved       0000 
        90: Root Status    00000000 pmep pmes ID:0 
    Enhanced Capabilities:
        100: CapID         0001 Advanced Error Reporting Capability
             Version       2
             NextPtr       148
        104: UncorrectableErrorStatus   00000000  dlpe sde ptlp fcpe ct ca mtlp ecrc ur acsv uie mcbtlp aeb tpb
        108: UncorrectableErrorMask     00400000  dlpe sde ptlp fcpe ct ca mtlp ecrc ur acsv UIE mcbtlp aeb tpb
        10c: UncorrectableErrorSeverity 00462030  DLPE SDE ptlp FCPE ct ca MTLP ecrc ur acsv UIE mcbtlp aeb tpb
        110: CorrectableErrorStatus     00000000  re btlp bdllp rnr rtt anfe cie hlo
        114: CorrectableErrorMask       00006000  re btlp bdllp rnr rtt ANFE CIE hlo
        118: CapabilitiesAndControl     00000000  FirstErr:0 ecrcgcap ecrcgen ecrcccap ecrccen mhrcap mhren tplp
        11c: HeaderLog:    00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  
        148: CapID         0018 Latency Tolerance Reporting (LTR) Capability
             Version       1
             NextPtr       150
             Latency       00000000 MaxSnoopValue:0 MaxSnoopScale:0 MaxNoSnoopValue:0 MaxNoSnoopScale:0
        150: CapID         001e L1 PM SS Capability
             Version       1
             NextPtr       000
```

To dump the raw data from device 1, do !pci f 1 0 0 0 160. It prints out raw data in address range 0 - 0x160

## References
1.  [www.lifewire.com/pci-express-pcie-2625962 ](https://www.lifewire.com/pci-express-pcie-2625962) 
2. Fun and Easy PCIe - How the PCIe protocol works -  [www.youtube.com/watch?v=sRx2YLzBIqk ](https://www.youtube.com/watch?v=sRx2YLzBIqk) 
3.  [docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-pci ](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-pci) 
4.   [en.wikipedia.org/wiki/PCI_configuration_space ](https://en.wikipedia.org/wiki/PCI_configuration_space) 
5.  [PCI_Express_Base_Specification_Revision_4.0.Ver.0.3.pdf](onenote:%23PCI%20Express%20Base%20Specification%20Rev4.0%20Ver0.3&section-id=%7B2a8f4eb1-309d-427b-8ef6-b20fe65a060b%7D&page-id=%7B3edc15cc-b948-44e0-8258-087bccc80f32%7D&end) 

  

