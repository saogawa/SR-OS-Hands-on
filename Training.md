## 目次
- [1. 初期設定](#1-初期設定)
  - [ハードウェア](#ハードウェア)
  - [NTP](#ntp)
  - [SNMP Trap](#snmp-trap)
  - [Syslog](#syslog)
  - [ユーザアカウント](#ユーザアカウント)
  - [ログインコントロール](#ログインコントロール)
  - [ターミナルロギング](#ターミナルロギング)
  - [systemループバック](#systemループバック)
- [2. 応用編](#2-応用編)
  - [物理ポート設定](#物理ポート設定)
  - [コア網側 : インターフェース設定](#コア網側--インターフェース設定)
  - [コア網側 : ISIS設定](#コア網側--isis設定)
  - [コア網側 : ISIS-SR(Segment-Routing)設定](#コア網側--isis-srsegment-routing設定)
  - [コア網側 : iBGP設定](#コア網側--ibgp設定)
  - [CE網側 : VPRN設定](#ce網側--vprn設定)
  - [疎通確認 (internet) : delayメトリック変更前](#疎通確認-internet--delayメトリック変更前)
  - [疎通確認(gamer) : delayメトリック変更前](#疎通確認gamer--delayメトリック変更前)
  - [Flex-Algo 設定](#flex-algo-設定)
  - [疎通確認 (internet) : delayメトリック変更後](#疎通確認-internet--delayメトリック変更後)
  - [疎通確認(gamer) : delayメトリック変更後](#疎通確認gamer--delayメトリック変更後)
  - [コア網側 : delayメトリックの変更](#コア網側--delayメトリックの変更)

# 1. 初期設定

## ハードウェア

---

・設定

```bash
(ex)[/]
A:admin@r1# admin show configuration /configure card 1
    card-type i24-800g-qsfpdd-1
    level he2800g+
    mda 1 {
        mda-type m24-800g-qsfpdd-1
    }
```

```bash
    /configure card 1 card-type i24-800g-qsfpdd-1
    /configure card 1 level he2800g+
    /configure card 1 mda 1 mda-type m24-800g-qsfpdd-1
```

・確認

```bash
(ex)[/]
A:admin@r1# show card

===============================================================================
Card Summary
===============================================================================
Slot      Provisioned Type                         Admin Operational   Comments
              Equipped Type (if different)         State State
-------------------------------------------------------------------------------
1         i24-800g-qsfpdd-1:he2800g+               up    up
A         cpm-1x                                   up    up/active
===============================================================================

(ex)[/]
A:admin@r1# show mda

===============================================================================
MDA Summary
===============================================================================
Slot  Mda   Provisioned Type                            Admin     Operational
                Equipped Type (if different)            State     State
-------------------------------------------------------------------------------
1     1     m24-800g-qsfpdd-1                           up        up
===============================================================================
```

## NTP

---

・設定

```bash
(ex)[/]
A:admin@r3# admin show configuration /configure system time
    zone {
        non-standard {
            name "jst"
            offset "09:00"
        }
    }
    ntp {
        admin-state enable
        peer 172.20.20.1 router-instance "management" {
            version 4
            prefer true
        }
	  }
```

```bash
    /configure system time zone non-standard name "jst"
    /configure system time zone non-standard offset "09:00"
    /configure system time ntp admin-state enable
    /configure system time ntp peer 172.20.20.1 router-instance "management" version 4
    /configure system time ntp peer 172.20.20.1 router-instance "management" prefer true
```

・確認

```bash
(ex)[/]
A:admin@r3# show system ntp peers

===============================================================================
NTP Active Associations
===============================================================================
State                     Reference ID    St Type  A  Poll Reach     Offset(ms)
    Router         Remote
-------------------------------------------------------------------------------
chosen                    162.159.200.1   4  actpr -  64   ...YYYYY  11.627
    management     172.20.20.1
===============================================================================

===============================================================================
NTP Clients
===============================================================================
vRouter                                                    Time Last Request Rx
    Address
-------------------------------------------------------------------------------
===============================================================================
```

## SNMP Trap

---

・設定

```bash
(ex)[/]
A:admin@r3# admin show configuration /configure log log-id 10
    source {
        main true
        security true
        change true
    }
    destination {
        snmp {
        }
    }

(ex)[/]
A:admin@r3# admin show configuration /configure log snmp-trap-group 10
    trap-target "snmptrapd" {
        address 172.20.20.1
        port 162
        version snmpv2c
        notify-community "public"
    }
```

```bash
    /configure log snmp-trap-group "10" trap-target "snmptrapd" address 172.20.20.1
    /configure log snmp-trap-group "10" trap-target "snmptrapd" port 162
    /configure log snmp-trap-group "10" trap-target "snmptrapd" version snmpv2c
    /configure log snmp-trap-group "10" trap-target "snmptrapd" notify-community "public"
    /configure log log-id "10" source main true
    /configure log log-id "10" source security true
    /configure log log-id "10" source change true
    /configure log log-id "10" destination { snmp }
```

・確認

```bash
(ex)[/]
A:admin@r3# admin save
Writing configuration to tftp://172.31.255.29/config.txt
Saving configuration OK
Completed.
```

```jsx
admin@pod1-KVM:~$ sudo tail -f /var/log/snmptrapd.log
[sudo] password for admin:
2024-08-29 11:05:58 localhost [UDP: [127.0.0.1]:35229->[127.0.0.1]:162]:
iso.3.6.1.2.1.1.3.0 = Timeticks: (9874407) 1 day, 3:25:44.07	iso.3.6.1.6.3.1.1.4.1.0 = OID: iso.3.6.1.4.1.311.1.1.3.1.2	iso.3.6.1.4.1.311.1.1.3.1.2 = STRING: "test trap test"
2024-08-29 11:28:16 clab-sr-r3 [UDP: [172.20.20.12]:162->[172.20.20.1]:162]:
iso.3.6.1.2.1.1.3.0 = Timeticks: (467673) 1:17:56.73	iso.3.6.1.6.3.1.1.4.1.0 = OID: iso.3.6.1.4.1.6527.3.1.3.1.0.11	iso.3.6.1.4.1.6527.3.1.2.1.7.1.0 = OID: iso.3.6.1.4.1.6527.3.1.2.12.5.1.2.10	iso.3.6.1.4.1.6527.3.1.2.1.7.2.0 = INTEGER: 2	iso.3.6.1.4.1.6527.3.1.2.1.7.3.0 = INTEGER: 3	iso.3.6.1.4.1.6527.3.1.2.1.7.9.0 = OID: iso.3.6.1.4.1.6527.3.1.2.12.5.1	iso.3.6.1.4.1.6527.3.1.2.1.7.19.0 = STRING: "Log Id 10"
```

## Syslog

---

・設定

```bash
(ex)[/]
A:admin@r3# admin show configuration /configure log syslog "1"
    address 172.20.20.1
    port 514

(ex)[/]
A:admin@r3# admin show configuration /configure log log-id 20
    source {
        main true
        security true
        change true
    }
    destination {
        syslog "1"
    }
```

```bash
    /configure log log-id "20" source main true
    /configure log log-id "20" source security true
    /configure log log-id "20" source change true
    /configure log log-id "20" destination syslog "1"
    /configure log syslog "1" address 172.20.20.1
    /configure log syslog "1" port 514
```

・確認

```bash
(ex)[/]
A:admin@r3# show log log-id

==============================================================================
Event Logs
==============================================================================
Name
Log  Source   Filter Admin Oper       Logged     Dropped Dest       Dest Size
Id            Id     State State                         Type       Id
------------------------------------------------------------------------------
10
 10  M S C    N/A    up    up             13          10 trap-group 10    100
20
 20  M S C    N/A    up    up             20           0 syslog     1     N/A
99
 99  M        N/A    up    up             84           0 memory           500
100
100  M        1001   up    up             11          73 memory           500
101
101  M S C    N/A    up    up            309           0 netconf          500
==============================================================================

```

```jsx
(ex)[/]
A:admin@r3# admin save
Writing configuration to tftp://172.31.255.29/config.txt
Saving configuration OK
Completed.
```

```jsx

admin@pod1-KVM:~$ sudo tail -f /var/log/syslog
Aug 29 02:38:11 172.31.255.30 TMNX: 11 Base SYSTEM-WARNING-ssiSaveConfigSucceeded-2002 [Configuration Save Succeeds]:  Configuration file saved to: tftp://172.31.255.29/config.txt
```

## ユーザアカウント

---

・設定

```bash
(ex)[/]
A:admin@r1# admin show configuration /configure system security user-params
    local-user {
        user "guest" {
            password "guest"
            restricted-to-home false
            access {
                console true
                ftp true
                ssh-cli true
            }
            console {
                member ["administrative"]
            }
        }
    }
```

```bash
    /configure system security user-params local-user user "guest" password "guest"
    /configure system security user-params local-user user "guest" restricted-to-home false
    /configure system security user-params local-user user "guest" access console true
    /configure system security user-params local-user user "guest" access ftp true
    /configure system security user-params local-user user "guest" access ssh-cli true
    /configure system security user-params local-user user "guest" console member ["administrative"]
```

・確認

```bash
[/]
A:admin2@r1# show system security user

===============================================================================
Users
===============================================================================
User ID      New Access                           Password Login   Failed Local
             Pwd Permissions                      Expires  Attempt Logins Conf
-------------------------------------------------------------------------------
admin        n   bt cc fp gr -- nc sp -- sc tc    never    9       0      y
guest        n   bt cc fp -- -- -- sp -- sc tc    never    2       0      y
-------------------------------------------------------------------------------
Number of users : 2
Permissions: (bt) Bluetooth, (cc) Console port CLI, (fp) FTP, (gr) gRPC,
             (li) LI, (nc) NETCONF, (sp) SCP/SFTP, (sn) SNMP, (sc) SSH CLI,
             (tc) Telnet CLI
===============================================================================
```

```bash
[/]
A:admin2@r1# logout
Connection to clab-sr-r1 closed.
root@pod1-KVM:/home/clab/sros-hands-on# ssh clab-sr-r1 -l guest
Warning: Permanently added 'clab-sr-r1' (RSA) to the list of known hosts.

guest@clab-sr-r1's password: guest

<SNIP>

[/]
A:guest@r1#
```

## ログインコントロール

---

・設定

```bash
(ex)[/]
A:admin@r1# admin show configuration /configure system security
    telnet-server true
    ftp-server true
<SNIP>

(ex)[/]
A:admin@r1# admin show configuration /configure system login-control
    idle-timeout none
    ssh {
        inbound-max-sessions 10
    }
    telnet {
        inbound-max-sessions 10
    }
```

```bash
    /configure system security telnet-server true
    /configure system security ftp-server true
    
    /configure system login-control idle-timeout none
    /configure system login-control ssh inbound-max-sessions 10
    /configure system login-control telnet inbound-max-sessions 10
```

・確認

```bash
(ex)[/]
A:admin@r1# show system security user

===============================================================================
Users
===============================================================================
User ID      New Access                           Password Login   Failed Local
             Pwd Permissions                      Expires  Attempt Logins Conf
-------------------------------------------------------------------------------
admin        n   bt cc fp gr -- nc sp -- sc tc    never    9       0      y
-------------------------------------------------------------------------------
Number of users : 1
Permissions: (bt) Bluetooth, (cc) Console port CLI, (fp) FTP, (gr) gRPC,
             (li) LI, (nc) NETCONF, (sp) SCP/SFTP, (sn) SNMP, (sc) SSH CLI,
             (tc) Telnet CLI
===============================================================================
```

```bash
(ex)[/]
A:admin@r1# show system security management

===============================================================================
Server Global
===============================================================================
Telnet:
Administrative State         : Enabled
Operational State            : Up
Telnet6:
Administrative State         : Disabled
Operational State            : Down
FTP:
Administrative State         : Enabled
Operational State            : Up
SSH:
Administrative State         : Enabled
Operational State            : Up
NETCONF:
Administrative State         : Enabled
Operational State            : Down
GRPC:
Administrative State         : Enabled
Operational State            : Up

===============================================================================
Server Router Instance [Base]
===============================================================================
Telnet:
Access allowed               : Allowed
Telnet6:
Access allowed               : Allowed
FTP:
Access allowed               : Allowed
SSH:
Access allowed               : Allowed
NETCONF:
Access allowed               : Allowed
GRPC:
Access allowed               : Allowed

===============================================================================
Server Router Instance [management]
===============================================================================
Telnet:
Access allowed               : Allowed
Telnet6:
Access allowed               : Allowed
FTP:
Access allowed               : Allowed
SSH:
Access allowed               : Allowed
NETCONF:
Access allowed               : Allowed
GRPC:
Access allowed               : Allowed
===============================================================================
```

## ターミナルロギング

・設定

```bash
(ex)[/]
A:admin@r1# admin show configuration /configure log log-id 30
    source {
        main true
        security true
        change true
    }
    destination {
        cli {
        }
    }
```

```bash
    /configure log log-id "30" source main true
    /configure log log-id "30" source security true
    /configure log log-id "30" source change true
    /configure log log-id "30" destination { cli }
```

・確認

```bash
(ex)[/configure log log-id "30" destination cli]
A:admin@r1# show log log-id

==============================================================================
Event Logs
==============================================================================
Name
Log  Source   Filter Admin Oper       Logged     Dropped Dest       Dest Size
Id            Id     State State                         Type       Id
------------------------------------------------------------------------------
30
 30  M S C    N/A    up    up              5           0 cli              100
99
 99  M        N/A    up    up            105           0 memory           500
100
100  M        1001   up    up             15          90 memory           500
101
101  M S C    N/A    up    up            318           0 netconf          500
==============================================================================
```

```bash
(ex)[/]
A:admin@r1# tools perform log subscribe-to log-id "30"

(ex)[/]
A:admin@r1#

(ex)[/]
A:admin@r1# tools perform log test-event

30 2024/08/29 04:12:20.217 UTC indeterminate: LOGGER #2011 Base Event Test
Test event has been generated with system object identifier tmnxModelSR1DDHFReg
System description: TiMOS-C-24.7.R1 cpm/x86_64 Nokia 7750 SR Copyright (c) 2000-2024 Nokia.
All rights reserved. All use subject to applicable license agreements.
Built on Thu Jul 11 15:05:03 PDT 2024 by builder in /builds/247B/R1/panos/main/sros

(ex)[/]
A:admin@r1# tools perform log unsubscribe-from log-id "30"
```

## systemループバック

---

・設定

```bash
(ex)[/]
A:admin@r1# admin show configuration /configure router interface "system"
    ipv4 {
        primary {
            address 192.0.2.1
            prefix-length 32
        }
    }
```

```bash
    /configure router "Base" interface "system" ipv4 primary address 192.0.2.1
    /configure router "Base" interface "system" ipv4 primary prefix-length 32
```

| ホスト名 | system loopback address (IPv4) | system loopback address (IPv6) |
| --- | --- | --- |
| clab-sr-r1 | 192.0.2.1 /32 | 192:2::1 /128 |
| clab-sr-r2 | 192.0.2.2 /32 | 192:2::2 /128 |
| clab-sr-r3 | 192.0.2.3 /32 | 192:2::3 /128 |
| clab-sr-r4 | 192.0.2.4 /32 | 192:2::4 /128 |
| clab-sr-r5 | 192.0.2.5 /32 | 192:2::5 /128 |
| clab-sr-r6 | 192.0.2.6 /32 | 192:2::6 /128 |

・確認

```bash
(ex)[/]
A:admin@r1# show router "Base" interface

===============================================================================
Interface Table (Router: Base)
===============================================================================
Interface-Name                   Adm       Opr(v4/v6)  Mode    Port/SapId
   IP-Address                                                  PfxState
-------------------------------------------------------------------------------
system                           Up        Up/Up       Network system
   192.0.2.1/32                                                n/a
   192:2::1/128                                                PREFERRED

<SNIP>
-------------------------------------------------------------------------------
Interfaces : 3
===============================================================================
```

# 2. 応用編

## 物理ポート設定

---

・設定

```bash
[/]
A:admin@r1# admin show configuration /configure port 1/1/c[1..3]
    port 1/1/c1 {
        admin-state enable
        connector {
            breakout c1-100g
        }
    }
    port 1/1/c2 {
        admin-state enable
        connector {
            breakout c1-100g
        }
    }
    port 1/1/c3 {
        admin-state enable
        connector {
            breakout c1-100g
        }
    }

[/]
A:admin@r1#

[/]
A:admin@r1# admin show configuration /configure port 1/1/c[1..3]/1
    port 1/1/c1/1 {
        admin-state enable
        ethernet {
            mode hybrid
            mtu 9800
        }
    }
    port 1/1/c2/1 {
        admin-state enable
        ethernet {
            mode hybrid
            mtu 9800
        }
    }
    port 1/1/c3/1 {
        admin-state enable
        ethernet {
            mode hybrid
            mtu 9800
        }
    }

```

```bash
    /configure port 1/1/c1 admin-state enable
    /configure port 1/1/c1 connector breakout c1-100g
    /configure port 1/1/c2 admin-state enable
    /configure port 1/1/c2 connector breakout c1-100g
    /configure port 1/1/c3 admin-state enable
    /configure port 1/1/c3 connector breakout c1-100g
    /configure port 1/1/c1/1 admin-state enable
    /configure port 1/1/c1/1 ethernet mode hybrid
    /configure port 1/1/c1/1 ethernet mtu 9800
    /configure port 1/1/c2/1 admin-state enable
    /configure port 1/1/c2/1 ethernet mode hybrid
    /configure port 1/1/c2/1 ethernet mtu 9800
    /configure port 1/1/c3/1 admin-state enable
    /configure port 1/1/c3/1 ethernet mode hybrid
    /configure port 1/1/c3/1 ethernet mtu 9800
```

・確認

```bash
[/]
A:admin@r1# show port

===============================================================================
Ports on Slot 1
===============================================================================
Port          Admin Link Port    Cfg  Oper LAG/ Port Port Port   C/QS/S/XFP/
Id            State      State   MTU  MTU  Bndl Mode Encp Type   MDIMDX
-------------------------------------------------------------------------------
1/1/c1        Up         Link Up                          conn   100G-CWDM4 2*
1/1/c1/1      Up    Yes  Up      9800 9800    - hybr dotq cgige
1/1/c2        Up         Link Up                          conn   100G-CWDM4 2*
1/1/c2/1      Up    Yes  Up      9800 9800    - hybr dotq cgige
1/1/c3        Up         Link Up                          conn   100G-CWDM4 2*
1/1/c3/1      Up    Yes  Up      9800 9800    - hybr dotq cgige
```

```bash

```

## コア網側 :  インターフェース設定

---

・設定

```bash
[/]
A:admin@r1# admin show configuration /configure router
    autonomous-system 65000
    interface "system" {
        ipv4 {
            primary {
                address 192.0.2.1
                prefix-length 32
            }
        }
        ipv6 {
            address 192:2::1 {
                prefix-length 128
            }
        }
    }
    interface "to_R3" {
        admin-state enable
        port 1/1/c2/1:0
        ipv4 {
            primary {
                address 192.168.13.0
                prefix-length 31
            }
        }
        if-attribute {
            delay {
                static 10000
            }
        }
    }
    interface "to_R4" {
        admin-state enable
        port 1/1/c3/1:0
        ipv4 {
            primary {
                address 192.168.14.0
                prefix-length 31
            }
        }
        if-attribute {
            delay {
                static 10000
            }
        }
    
    }
```

```bash
    /configure router "Base" autonomous-system 65000
    /configure router "Base" interface "system" ipv4 primary address 192.0.2.1
    /configure router "Base" interface "system" ipv4 primary prefix-length 32
    /configure router "Base" interface "system" ipv6 address 192:2::1 prefix-length 128
    /configure router "Base" interface "to_R3" admin-state enable
    /configure router "Base" interface "to_R3" port 1/1/c2/1:0
    /configure router "Base" interface "to_R3" ipv4 primary address 192.168.13.0
    /configure router "Base" interface "to_R3" ipv4 primary prefix-length 31
    /configure router "Base" interface "to_R3" if-attribute delay static 10000
    /configure router "Base" interface "to_R4" admin-state enable
    /configure router "Base" interface "to_R4" port 1/1/c3/1:0
    /configure router "Base" interface "to_R4" ipv4 primary address 192.168.14.0
    /configure router "Base" interface "to_R4" ipv4 primary prefix-length 31
    /configure router "Base" interface "to_R4" if-attribute delay static 10000
```

・確認

```bash
[/]
A:admin@r1# show router "Base" interface

===============================================================================
Interface Table (Router: Base)
===============================================================================
Interface-Name                   Adm       Opr(v4/v6)  Mode    Port/SapId
   IP-Address                                                  PfxState
-------------------------------------------------------------------------------
system                           Up        Up/Up       Network system
   192.0.2.1/32                                                n/a
   192:2::1/128                                                PREFERRED
to_R3                            Up        Up/Down     Network 1/1/c2/1:0
   192.168.13.0/31                                             n/a
to_R4                            Up        Up/Down     Network 1/1/c3/1:0
   192.168.14.0/31                                             n/a
-------------------------------------------------------------------------------
Interfaces : 3
===============================================================================
```

## コア網側 :  ISIS設定

---

・設定

```bash
[/]
A:admin@r1# admin show configuration /configure router "Base" isis
    admin-state enable
    advertise-router-capability as
    level-capability 2
    traffic-engineering false
    area-address [49.0001]
    flexible-algorithms {
        admin-state enable
        flex-algo 128 {
            participate true
            advertise "Flex-Algo-128"
        }
    }
    traffic-engineering-options {
        application-link-attributes {
        }
    }
    segment-routing {
        admin-state enable
        prefix-sid-range {
            global
        }
    }
    interface "system" {
        ipv4-node-sid {
            index 1
        }
        flex-algo 128 {
            ipv4-node-sid {
                index 11
            }
        }
    }
    interface "to_R3" {
        interface-type point-to-point
        level 1 {
        }
    }
    interface "to_R4" {
        interface-type point-to-point
        level 1 {
        }
    }
    level 2 {
        wide-metrics-only true
    }
```

```bash
    /configure router "Base" isis 0 admin-state enable
    /configure router "Base" isis 0 advertise-router-capability as
    /configure router "Base" isis 0 level-capability 2
    /configure router "Base" isis 0 interface "to_R3" interface-type point-to-point
    /configure router "Base" isis 0 interface "to_R3" { level 1 }
    /configure router "Base" isis 0 interface "to_R4" interface-type point-to-point
    /configure router "Base" isis 0 interface "to_R4" { level 1 }
    /configure router "Base" isis 0 level 2 wide-metrics-only true
```

・確認

```bash
[/]
A:admin@r1# show router "Base" isis interface

===============================================================================
Rtr Base ISIS Instance 0 Interfaces
===============================================================================
Interface                        Level CircID  Oper      L1/L2 Metric     Type
                                               State
-------------------------------------------------------------------------------
system                           L1L2  1       Up        0/0              p2p
to_R3                            L1L2  2       Up        10/10            p2p
to_R4                            L1L2  3       Up        10/10            p2p
-------------------------------------------------------------------------------
Interfaces : 3
===============================================================================
```

```bash
[/]
A:admin@r1# show router "Base" isis adjacency

===============================================================================
Rtr Base ISIS Instance 0 Adjacency
===============================================================================
System ID                Usage State Hold Interface                     MT-ID
-------------------------------------------------------------------------------
r3                       L2    Up    19   to_R3                         0
r4                       L2    Up    19   to_R4                         0
-------------------------------------------------------------------------------
Adjacencies : 2
===============================================================================
```

## コア網側 :  ISIS-SR(Segment-Routing)設定

---

・設定

```bash
[/]
A:admin@r1# admin show configuration /configure router "Base" isis
    flexible-algorithms {
        admin-state enable
        flex-algo 128 {
            participate true
            advertise "Flex-Algo-128"
        }
    }
    traffic-engineering-options {
        application-link-attributes {
        }
    }
    segment-routing {
        admin-state enable
        prefix-sid-range {
            global
        }
    }
    interface "system" {
        ipv4-node-sid {
            index 1
        }
        flex-algo 128 {
            ipv4-node-sid {
                index 11
            }
        }
    }
```

```bash
    /configure router "Base" isis 0 interface "system" ipv4-node-sid index 1
    /configure router "Base" isis 0 interface "system" flex-algo 128 ipv4-node-sid index 11    
    /configure router "Base" isis 0 traffic-engineering false
    /configure router "Base" isis 0 flexible-algorithms admin-state enable
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 participate true
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 advertise "Flex-Algo-128"
    /configure router "Base" isis 0 traffic-engineering-options { application-link-attributes }
    /configure router "Base" isis 0 segment-routing admin-state enable
    /configure router "Base" isis 0 segment-routing prefix-sid-range global
```

・確認

```bash
[/]
A:admin@r1# show router "Base" isis flex-algo

===============================================================================
Rtr Base ISIS Instance 0 Flex-Algos
===============================================================================
Flex-Algo Level Advertising Participating Num       Selected FAD
                                          FADs      Owner (System-Id)
-------------------------------------------------------------------------------
128       L2    Yes         Yes           1         1920.0000.2001
-------------------------------------------------------------------------------
FAD: Flexible Algorithm Definition
-------------------------------------------------------------------------------
No. of Flex-Algos: 1 (1 unique)
===============================================================================
```

```bash
[/]
A:admin@r1# show router "Base" isis sid-stats adj

===============================================================================
Rtr Base ISIS Instance 0 Sid Statistics
===============================================================================
Ingress Label     : 524284              Type              : adjacency
Prefix            : 192.168.14.1/32
Interface         : to_R4
Ingress Oper State: disabled            Egress Oper State : disabled
Ingress Octets    : 0                   Egress Octets     : 0
Ingress Packets   : 0                   Egress Packets    : 0

Ingress Label     : 524285              Type              : adjacency
Prefix            : 192.168.13.1/32
Interface         : to_R3
Ingress Oper State: disabled            Egress Oper State : disabled
Ingress Octets    : 0                   Egress Octets     : 0
Ingress Packets   : 0                   Egress Packets    : 0

-------------------------------------------------------------------------------
Sid count : 2
===============================================================================
```

```bash
[/]
A:admin@r1# show router "Base" isis sid-stats adj

===============================================================================
Rtr Base ISIS Instance 0 Sid Statistics
===============================================================================
Ingress Label     : 524284              Type              : adjacency
Prefix            : 192.168.14.1/32
Interface         : to_R4
Ingress Oper State: disabled            Egress Oper State : disabled
Ingress Octets    : 0                   Egress Octets     : 0
Ingress Packets   : 0                   Egress Packets    : 0

Ingress Label     : 524285              Type              : adjacency
Prefix            : 192.168.13.1/32
Interface         : to_R3
Ingress Oper State: disabled            Egress Oper State : disabled
Ingress Octets    : 0                   Egress Octets     : 0
Ingress Packets   : 0                   Egress Packets    : 0

-------------------------------------------------------------------------------
Sid count : 2
===============================================================================
```

## コア網側 :  iBGP設定

---

・設定

```bash
[/]
A:admin@r1# admin show configuration /configure router "Base" bgp
    admin-state enable
    group "iBGP" {
        peer-as 65000
        family {
            vpn-ipv4 true
        }
    }
    neighbor "192.0.2.3" {
        group "iBGP"
    }
```

```bash
    /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp group "iBGP" peer-as 65000
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp neighbor "192.0.2.3" group "iBGP"
```

・確認

```bash
[/]
A:admin@r1# show router "Base" bgp neighbor

===============================================================================
BGP Neighbor
===============================================================================
-------------------------------------------------------------------------------
Peer                 : 192.0.2.3
Description          : (Not Specified)
Group                : iBGP
-------------------------------------------------------------------------------
Peer AS              : 65000            Peer Port            : 179
Peer Address         : 192.0.2.3
Local AS             : 65000            Local Port           : 50869
Local Address        : 192.0.2.1
Peer Type            : Internal         Dynamic Peer         : No
State                : Established      Last State           : Active

```

```bash
*(ex)[/]
A:admin@r1# show router bgp neighbor 192.0.2.3 received-routes vpn-ipv4
===============================================================================
 BGP Router ID:192.0.2.1        AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP VPN-IPv4 Routes
===============================================================================
Flag  Network                                            LocalPref   MED
      Nexthop (Router)                                   Path-Id     IGP Cost
      As-Path                                                        Label
-------------------------------------------------------------------------------
u*>i  1:2:10.0.2.0/24                                    100         None
      192.0.2.2                                          None        30
      No As-Path                                                     524286
u*>i  1:2:20.0.2.0/24                                    100         None
      192.0.2.2                                          None        30
      No As-Path                                                     524286
-------------------------------------------------------------------------------
Routes : 2
===============================================================================
```

## CE網側 :  VPRN設定

---

・設定

```bash
*(ex)[/]
A:admin@r1# admin show configuration /configure service vprn "customer1"
    admin-state enable
    service-id 1
    customer "1"
    bgp-ipvpn {
        mpls {
            admin-state enable
            route-distinguisher "1:1"
            vrf-target {
                community "target:65000:1"
            }
            vrf-import {
                policy ["customer1-import"]
            }
            auto-bind-tunnel {
                resolution filter
                allow-flex-algo-fallback true
                resolution-filter {
                    sr-isis true
                }
            }
        }
    }
    interface "to_client1" {
        ipv4 {
            primary {
                address 10.0.1.1
                prefix-length 24
            }
        }
        sap 1/1/c1/1:10 {
        }
    }
    interface "to_gamer1" {
        ipv4 {
            primary {
                address 20.0.1.1
                prefix-length 24
            }
        }
        sap 1/1/c1/1:20 {
        }
    }
```

```bash
    /configure service vprn "customer1" admin-state enable
    /configure service vprn "customer1" service-id 1
    /configure service vprn "customer1" customer "1"
    /configure service vprn "customer1" bgp-ipvpn mpls admin-state enable
    /configure service vprn "customer1" bgp-ipvpn mpls route-distinguisher "1:1"
    /configure service vprn "customer1" bgp-ipvpn mpls vrf-target community "target:65000:1"
    /configure service vprn "customer1" bgp-ipvpn mpls vrf-import policy ["customer1-import"]
    /configure service vprn "customer1" bgp-ipvpn mpls auto-bind-tunnel resolution filter
    /configure service vprn "customer1" bgp-ipvpn mpls auto-bind-tunnel allow-flex-algo-fallback true
    /configure service vprn "customer1" bgp-ipvpn mpls auto-bind-tunnel resolution-filter sr-isis true
    /configure service vprn "customer1" interface "to_client1" ipv4 primary address 10.0.1.1
    /configure service vprn "customer1" interface "to_client1" ipv4 primary prefix-length 24
    /configure service vprn "customer1" interface "to_client1" { sap 1/1/c1/1:10 }
    /configure service vprn "customer1" interface "to_gamer1" ipv4 primary address 20.0.1.1
    /configure service vprn "customer1" interface "to_gamer1" ipv4 primary prefix-length 24
    /configure service vprn "customer1" interface "to_gamer1" { sap 1/1/c1/1:20 }
```

・確認

```bash
*(ex)[/]
A:admin@r1# show router 1 interface

===============================================================================
Interface Table (Service: 1)
===============================================================================
Interface-Name                   Adm       Opr(v4/v6)  Mode    Port/SapId
   IP-Address                                                  PfxState
-------------------------------------------------------------------------------
to_client1                       Up        Up/Down     VPRN    1/1/c1/1:10
   10.0.1.1/24                                                 n/a
to_gamer1                        Up        Up/Down     VPRN    1/1/c1/1:20
   20.0.1.1/24                                                 n/a
-------------------------------------------------------------------------------
Interfaces : 2
===============================================================================

*(ex)[/]
A:admin@r1# show router 1 route-table

===============================================================================
Route Table (Service: 1)
===============================================================================
Dest Prefix[Flags]                            Type    Proto     Age        Pref
      Next Hop[Interface Name]                                    Metric
-------------------------------------------------------------------------------
10.0.1.0/24                                   Local   Local     04h56m45s  0
       to_client1                                                   0
10.0.2.0/24                                   Remote  BGP VPN   04h55m26s  170
       192.0.2.2 (tunneled:SR-ISIS:524299)                          30
20.0.1.0/24                                   Local   Local     04h56m45s  0
       to_gamer1                                                    0
20.0.2.0/24                                   Remote  BGP VPN   04h55m26s  170
       192.0.2.2 (tunneled:SR-ISIS:524296)                          30000
-------------------------------------------------------------------------------
No. of Routes: 4
Flags: n = Number of times nexthop is repeated
       B = BGP backup route available
       L = LFA nexthop available
       S = Sticky ECMP requested
===============================================================================
```

## 疎通確認 (internet)  : delayメトリック変更前

---

・確認

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh start internet
Starting non-gamer traffic to internet
Connecting to host 10.0.2.10, port 5201
```

```bash
[/]
A:admin@r1# monitor port 1/1/c2/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c2/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<SNIP>

-------------------------------------------------------------------------------
At time t = 24 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                              5721                 257064
Packets                                               73                    170
Errors                                                 0                      0
Bits                                               45768                2056512
Utilization (% of port capacity)                   ~0.00                  ~0.00
```

```bash
[/]
A:admin@r1# monitor port 1/1/c3/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c3/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<SNIP>

-------------------------------------------------------------------------------
At time t = 6 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                                 0                     24
Packets                                                0                      0
Errors                                                 0                      0
Bits                                                   0                    192
Utilization (% of port capacity)                    0.00                  ~0.00
```

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh stop internet
Stopping traffic
```

## 疎通確認(gamer) : delayメトリック変更前

---

・確認

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh start gamer
Starting gamer traffic
Connecting to host 20.0.2.10, port 5202
```

```bash
[/]
A:admin@r1# monitor port 1/1/c2/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c2/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<snip>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                              6524                 270260
Packets                                               83                    178
Errors                                                 0                      0
Bits                                               52192                2162080
Utilization (% of port capacity)                   ~0.00                  ~0.00
```

```bash
[/]
A:admin@r1# monitor port 1/1/c3/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c3/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                                 0                      0
Packets                                                0                      0
Errors                                                 0                      0
Bits                                                   0                      0
Utilization (% of port capacity)                    0.00                   0.00
```

```bash
root@pod1-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh stop gamer
Stopping traffic
```

## Flex-Algo  設定

---

・設定

```bash

[/show router isis segment-routing-v6]
A:admin@r1# admin show configuration /configure routing-options
    flexible-algorithm-definitions {
        flex-algo "Flex-Algo-128" {
            admin-state enable
            description "Flex-Algo for Delay Metric"
            metric-type delay
        }
    }

[/]
A:admin@r1# admin show configuration /configure router "Base" isis
    }
    flexible-algorithms {
        admin-state enable
        flex-algo 128 {
            participate true
            advertise "Flex-Algo-128"
        }
    }
    traffic-engineering-options {
        application-link-attributes {
        }
    }
    segment-routing {
        admin-state enable
        prefix-sid-range {
            global
        }
    }
    interface "system" {
        ipv4-node-sid {
            index 1
        }
        flex-algo 128 {
            ipv4-node-sid {
                index 11
            }
        }
   
   
  
```

```bash
[/]
A:admin@r1# admin show configuration /configure service vprn "customer1"
    admin-state enable
    service-id 1
    customer "1"
    bgp-ipvpn {
        mpls {
            admin-state enable
            route-distinguisher "1:1"
            vrf-target {
                community "target:65000:1"
            }
            vrf-import {
                policy ["customer1-import"]
            }
            auto-bind-tunnel {
                resolution filter
                allow-flex-algo-fallback true
                resolution-filter {
                    sr-isis true
                }
            }
        }
    }

```

```bash
    /configure service vprn "customer1" admin-state enable
    /configure service vprn "customer1" service-id 1
    /configure service vprn "customer1" customer "1"
    /configure service vprn "customer1" bgp-ipvpn mpls admin-state enable
    /configure service vprn "customer1" bgp-ipvpn mpls route-distinguisher "1:1"
    /configure service vprn "customer1" bgp-ipvpn mpls vrf-target community "target:65000:1"
    /configure service vprn "customer1" bgp-ipvpn mpls vrf-import policy ["customer1-import"]
    /configure service vprn "customer1" bgp-ipvpn mpls auto-bind-tunnel resolution filter
    /configure service vprn "customer1" bgp-ipvpn mpls auto-bind-tunnel allow-flex-algo-fallback true
    /configure service vprn "customer1" bgp-ipvpn mpls auto-bind-tunnel resolution-filter sr-isis true
    /configure router "Base" isis 0 flexible-algorithms admin-state enable
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 participate true
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 advertise "Flex-Algo-128"
    /configure router "Base" isis 0 traffic-engineering-options { application-link-attributes }
    /configure router "Base" isis 0 segment-routing admin-state enable
    /configure router "Base" isis 0 segment-routing prefix-sid-range global
    /configure router "Base" isis 0 interface "system" ipv4-node-sid index 1
    /configure router "Base" isis 0 interface "system" flex-algo 128 ipv4-node-sid index 11
    
    
```

・確認

```bash
[/]
A:admin@r1# show router "Base" isis flex-algo

===============================================================================
Rtr Base ISIS Instance 0 Flex-Algos
===============================================================================
Flex-Algo Level Advertising Participating Num       Selected FAD
                                          FADs      Owner (System-Id)
-------------------------------------------------------------------------------
128       L2    Yes         Yes           1         1920.0000.2001
-------------------------------------------------------------------------------
FAD: Flexible Algorithm Definition
-------------------------------------------------------------------------------
No. of Flex-Algos: 1 (1 unique)
===============================================================================
```

```bash

```

## 疎通確認 (internet)  : delayメトリック変更前

---

・確認

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh start internet
Starting non-gamer traffic to internet
Connecting to host 10.0.2.10, port 5201
```

```bash
[/]
A:admin@r1# monitor port 1/1/c2/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c2/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                              6367                 268458
Packets                                               82                    177
Errors                                                 0                      0
Bits                                               50936                2147664
Utilization (% of port capacity)                   ~0.00                  ~0.00
```

```bash
[/]
A:admin@r1# monitor port 1/1/c3/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c3/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                                51                      0
Packets                                                0                      0
Errors                                                 0                      0
Bits                                                 408                      0
Utilization (% of port capacity)                   ~0.00                   0.00
```

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh stop internet
Stopping traffic
```

## 疎通確認(gamer) : delayメトリック変更前

---

・確認

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh start gamer
Starting gamer traffic
Connecting to host 20.0.2.10, port 5202
```

```bash
[/]
A:admin@r1# monitor port 1/1/c2/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c2/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<snip>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                              6524                 270260
Packets                                               83                    178
Errors                                                 0                      0
Bits                                               52192                2162080
Utilization (% of port capacity)                   ~0.00                  ~0.00
```

```bash
[/]
A:admin@r1# monitor port 1/1/c3/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c3/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------

<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                                 0                      0
Packets                                                0                      0
Errors                                                 0                      0
Bits                                                   0                      0
Utilization (% of port capacity)                    0.00                   0.00
```

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh stop gamer
Stopping traffic
```

## コア網側 :  delayメトリックの変更

---

・設定

```bash
(ex)[/]
A:admin@r5# admin show configuration /configure router interface "to_R5"
    if-attribute {
        delay {
            static 50000
        }
    }
```

・確認

```bash
(ex)[/]
A:admin@r3# admin show configuration /configure router interface "to_R5"
    if-attribute {
        delay {
            static 50000
        }
    }
```

```bash

    /configure router "Base" interface "to_R5" if-attribute delay static 50000
```

## 疎通確認(internet) : delayメトリック変更後

---

・設定

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh start internet
Starting non-gamer traffic to internet
Connecting to host 10.0.2.10, port 5201
```

・確認

```bash
[/]
A:admin@r1# monitor port 1/1/c2/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c2/1
===============================================================================
                                                   Input                 Output
<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                              5051                 230946
Packets                                               65                    152
Errors                                                 0                      0
Bits                                               40408                1847568
Utilization (% of port capacity)                   ~0.00                  ~0.00
```

```bash
[/]
A:admin@r1# monitor port 1/1/c3/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c3/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------
<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                                 0                     37
Packets                                                0                      1
Errors                                                 0                      0
Bits                                                   0                    296
Utilization (% of port capacity)                    0.00                  ~0.00
```

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh stop internet
Stopping traffic
```

## 疎通確認(gamer) : delayメトリック変更後

---

・設定

```bash
root@pod1-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh start gamer
Starting gamer traffic
Connecting to host 20.0.2.10, port 5202
```

・確認

```bash
[/]
A:admin@r1# monitor port 1/1/c2/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c2/1
===============================================================================
                                                   Input                 Output
<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                              5928                      0
Packets                                               76                      0
Errors                                                 0                      0
Bits                                               47424                      0
Utilization (% of port capacity)                   ~0.00                   0.00
```

```bash
[/]
A:admin@r1# monitor port 1/1/c3/1 rate interval 3

===============================================================================
Monitor statistics for Port 1/1/c3/1
===============================================================================
                                                   Input                 Output
-------------------------------------------------------------------------------
<SNIP>

-------------------------------------------------------------------------------
At time t = 3 sec (Mode: Rate)
-------------------------------------------------------------------------------
Octets                                                51                 258561
Packets                                                0                    170
Errors                                                 0                      0
Bits                                                 408                2068488
Utilization (% of port capacity)                   ~0.00                  ~0.00
```

```bash
root@pod5-KVM:/home/clab/sros-hands-on# /home/clab/sros-hands-on/traffic.sh stop gamer
Stopping traffic
```
