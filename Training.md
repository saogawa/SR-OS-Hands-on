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

```bash
admin@DL360-G10-006:~$ sudo tcpdump -n -i any udp port 162 or udp port 514
[sudo] password for admin: admin123
tcpdump: data link type LINUX_SLL2
```

```bash
(ex)[/ ]
A:admin@r1# tools perform log test-event
```

```bash
admin@DL360-G10-006:~$ sudo tcpdump -n -i any udp port 162 or udp port 514
[sudo] password for admin:
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
08:45:18.743666 veth028e238 P   IP 172.20.20.4.162 > 172.20.20.1.162:  V2Trap(348)  .1.3.6.1.2.1.1.3.0=1973302 .1.3.6.1.6.3.1.1.4.1.0=.1.3.6.1.4.1.6527.3.1.3.12.0.6 .1.3.6.1.2.1.1.1.0=54_69_4d_4f_53_2d_43_2d_32_34_2e_37_2e_52_31_20_63_70_6d_2f_78_38_36_5f_36_34_20_4e_6f_6b_69_61_20_37_37_35_30_20_53_52_20_43_6f_70_79_72_69_67_68_74_20_28_63_29_20_32_30_30_30_2d_32_30_32_34_20_4e_6f_6b_69_61_2e_0d_0a_41_6c_6c_20_72_69_67_68_74_73_20_72_65_73_65_72_76_65_64_2e_20_41_6c_6c_20_75_73_65_20_73_75_62_6a_65_63_74_20_74_6f_20_61_70_70_6c_69_63_61_62_6c_65_20_6c_69_63_65_6e_73_65_20_61_67_72_65_65_6d_65_6e_74_73_2e_0d_0a_42_75_69_6c_74_20_6f_6e_20_54_68_75_20_4a_75_6c_20_31_31_20_31_35_3a_30_35_3a_30_33_20_50_44_54_20_32_30_32_34_20_62_79_20_62_75_69_6c_64_65_72_20_69_6e_20_2f_62_75_69_6c_64_73_2f_32_34_37_42_2f_52_31_2f_70_61_6e_6f_73_2f_6d_61_69_6e_2f_73_72_6f_73_0d_0a .1.3.6.1.2.1.1.2.0=.1.3.6.1.4.1.6527.1.3.35 .1.3.6.1.4.1.6527.3.1.2.12.35.0=""
08:45:18.743666 br-2df8edd81fb1 In  IP 172.20.20.4.162 > 172.20.20.1.162:  V2Trap(348)  .1.3.6.1.2.1.1.3.0=1973302 .1.3.6.1.6.3.1.1.4.1.0=.1.3.6.1.4.1.6527.3.1.3.12.0.6 .1.3.6.1.2.1.1.1.0=54_69_4d_4f_53_2d_43_2d_32_34_2e_37_2e_52_31_20_63_70_6d_2f_78_38_36_5f_36_34_20_4e_6f_6b_69_61_20_37_37_35_30_20_53_52_20_43_6f_70_79_72_69_67_68_74_20_28_63_29_20_32_30_30_30_2d_32_30_32_34_20_4e_6f_6b_69_61_2e_0d_0a_41_6c_6c_20_72_69_67_68_74_73_20_72_65_73_65_72_76_65_64_2e_20_41_6c_6c_20_75_73_65_20_73_75_62_6a_65_63_74_20_74_6f_20_61_70_70_6c_69_63_61_62_6c_65_20_6c_69_63_65_6e_73_65_20_61_67_72_65_65_6d_65_6e_74_73_2e_0d_0a_42_75_69_6c_74_20_6f_6e_20_54_68_75_20_4a_75_6c_20_31_31_20_31_35_3a_30_35_3a_30_33_20_50_44_54_20_32_30_32_34_20_62_79_20_62_75_69_6c_64_65_72_20_69_6e_20_2f_62_75_69_6c_64_73_2f_32_34_37_42_2f_52_31_2f_70_61_6e_6f_73_2f_6d_61_69_6e_2f_73_72_6f_73_0d_0a .1.3.6.1.2.1.1.2.0=.1.3.6.1.4.1.6527.1.3.35 .1.3.6.1.4.1.6527.3.1.2.12.35.0=""
08:45:18.743734 veth028e238 P   IP 172.20.20.4.514 > 172.20.20.1.514: SYSLOG local7.info, length: 435
08:45:18.743734 br-2df8edd81fb1 In  IP 172.20.20.4.514 > 172.20.20.1.514: SYSLOG local7.info, length: 435
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

```bash
admin@DL360-G10-006:~$ sudo tcpdump -n -i any udp port 162 or udp port 514
[sudo] password for admin: admin123
tcpdump: data link type LINUX_SLL2
```

```bash
(ex)[/ ]
A:admin@r1# tools perform log test-event
```

```bash
admin@DL360-G10-006:~$ sudo tcpdump -n -i any udp port 162 or udp port 514
[sudo] password for admin:
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
08:45:18.743666 veth028e238 P   IP 172.20.20.4.162 > 172.20.20.1.162:  V2Trap(348)  .1.3.6.1.2.1.1.3.0=1973302 .1.3.6.1.6.3.1.1.4.1.0=.1.3.6.1.4.1.6527.3.1.3.12.0.6 .1.3.6.1.2.1.1.1.0=54_69_4d_4f_53_2d_43_2d_32_34_2e_37_2e_52_31_20_63_70_6d_2f_78_38_36_5f_36_34_20_4e_6f_6b_69_61_20_37_37_35_30_20_53_52_20_43_6f_70_79_72_69_67_68_74_20_28_63_29_20_32_30_30_30_2d_32_30_32_34_20_4e_6f_6b_69_61_2e_0d_0a_41_6c_6c_20_72_69_67_68_74_73_20_72_65_73_65_72_76_65_64_2e_20_41_6c_6c_20_75_73_65_20_73_75_62_6a_65_63_74_20_74_6f_20_61_70_70_6c_69_63_61_62_6c_65_20_6c_69_63_65_6e_73_65_20_61_67_72_65_65_6d_65_6e_74_73_2e_0d_0a_42_75_69_6c_74_20_6f_6e_20_54_68_75_20_4a_75_6c_20_31_31_20_31_35_3a_30_35_3a_30_33_20_50_44_54_20_32_30_32_34_20_62_79_20_62_75_69_6c_64_65_72_20_69_6e_20_2f_62_75_69_6c_64_73_2f_32_34_37_42_2f_52_31_2f_70_61_6e_6f_73_2f_6d_61_69_6e_2f_73_72_6f_73_0d_0a .1.3.6.1.2.1.1.2.0=.1.3.6.1.4.1.6527.1.3.35 .1.3.6.1.4.1.6527.3.1.2.12.35.0=""
08:45:18.743666 br-2df8edd81fb1 In  IP 172.20.20.4.162 > 172.20.20.1.162:  V2Trap(348)  .1.3.6.1.2.1.1.3.0=1973302 .1.3.6.1.6.3.1.1.4.1.0=.1.3.6.1.4.1.6527.3.1.3.12.0.6 .1.3.6.1.2.1.1.1.0=54_69_4d_4f_53_2d_43_2d_32_34_2e_37_2e_52_31_20_63_70_6d_2f_78_38_36_5f_36_34_20_4e_6f_6b_69_61_20_37_37_35_30_20_53_52_20_43_6f_70_79_72_69_67_68_74_20_28_63_29_20_32_30_30_30_2d_32_30_32_34_20_4e_6f_6b_69_61_2e_0d_0a_41_6c_6c_20_72_69_67_68_74_73_20_72_65_73_65_72_76_65_64_2e_20_41_6c_6c_20_75_73_65_20_73_75_62_6a_65_63_74_20_74_6f_20_61_70_70_6c_69_63_61_62_6c_65_20_6c_69_63_65_6e_73_65_20_61_67_72_65_65_6d_65_6e_74_73_2e_0d_0a_42_75_69_6c_74_20_6f_6e_20_54_68_75_20_4a_75_6c_20_31_31_20_31_35_3a_30_35_3a_30_33_20_50_44_54_20_32_30_32_34_20_62_79_20_62_75_69_6c_64_65_72_20_69_6e_20_2f_62_75_69_6c_64_73_2f_32_34_37_42_2f_52_31_2f_70_61_6e_6f_73_2f_6d_61_69_6e_2f_73_72_6f_73_0d_0a .1.3.6.1.2.1.1.2.0=.1.3.6.1.4.1.6527.1.3.35 .1.3.6.1.4.1.6527.3.1.2.12.35.0=""
08:45:18.743734 veth028e238 P   IP 172.20.20.4.514 > 172.20.20.1.514: SYSLOG local7.info, length: 435
08:45:18.743734 br-2df8edd81fb1 In  IP 172.20.20.4.514 > 172.20.20.1.514: SYSLOG local7.info, length: 435
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
(gl)[/]
A:admin@r1# admin show configuration /configure policy-options
    community "customer1-export" {
        member "target:65000:1" { }
    }
    community "customer1-import" {
        member "target:65000:1" { }
    }
    prefix-list "gaming" {
        prefix 20.0.2.0/24 type exact {
        }
    }
    prefix-list "internet" {
        prefix 10.0.2.0/24 type exact {
        }
    }
    policy-statement "customer1-import" {
        entry 10 {
            from {
                prefix-list ["internet"]
                community {
                    name "customer1-import"
                }
            }
            action {
                action-type accept
            }
        }
        entry 20 {
            from {
                prefix-list ["gaming"]
                community {
                    name "customer1-import"
                }
            }
            action {
                action-type accept
                flex-algo 128
            }
        }
        default-action {
            action-type reject
        }
    }
```

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
    /configure policy-options community "customer1-export" { member "target:65000:1" }
    /configure policy-options community "customer1-import" { member "target:65000:1" }
    /configure policy-options prefix-list "gaming" { prefix 20.0.2.0/24 type exact }
    /configure policy-options prefix-list "internet" { prefix 10.0.2.0/24 type exact }
    /configure policy-options policy-statement "customer1-import" entry 10 from prefix-list ["internet"]
    /configure policy-options policy-statement "customer1-import" entry 10 from community name "customer1-import"
    /configure policy-options policy-statement "customer1-import" entry 10 action action-type accept
    /configure policy-options policy-statement "customer1-import" entry 20 from prefix-list ["gaming"]
    /configure policy-options policy-statement "customer1-import" entry 20 from community name "customer1-import"
    /configure policy-options policy-statement "customer1-import" entry 20 action action-type accept
    /configure policy-options policy-statement "customer1-import" entry 20 action flex-algo 128
    /configure policy-options policy-statement "customer1-import" default-action action-type reject
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
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" admin-state enable
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" description "Flex-Algo for Delay Metric"
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" metric-type delay
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
