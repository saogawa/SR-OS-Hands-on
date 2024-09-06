## 目次

1. [初期設定](#初期設定)
   - [ハードウェア](#ハードウェア)
   - [NTP](#NTP)
   - [SNMP Trap](#SNMP-Trap)
   - [Syslog](#Syslog)
   - [ユーザアカウント](#ユーザアカウント)
   - [ログインコントロール](#ログインコントロール)
   - [ターミナルロギング](#ターミナルロギング)
   - [systemループバック](#systemループバック)
2. [応用編](#応用編)
   - [物理ポート設定](#物理ポート設定)
   - [コア網側インターフェース設定](#コア網側インターフェース設定)
   - [コア網側ISIS設定](#コア網側ISIS設定)
   - [コア網側ISIS-SR設定](#コア網側ISIS-SR設定)
   - [コア網側iBGP設定](#コア網側iBGP設定)
   - [CE網側設定_カスタマー情報](#CE網側設定_カスタマー情報)
   - [CE網側設定_EVPN_L3VPN](#CE網側設定_EVPN_L3VPN)
   - [CE網側設定_EVPN_L2VPN_ELAN](#CE網側設定_EVPN_L2VPN_ELAN)
   - [CE網側設定_EVPN_L2VPN_VPWS](#CE網側設定_EVPN_L2VPN_VPWS)
   - [疎通確認_internet_delayメトリック変更前](#疎通確認_internet_delayメトリック変更前)
   - [疎通確認_gamer_delayメトリック変更前](#疎通確認_gamer_delayメトリック変更前)
   - [コア網側_R3-R5_delayメトリックの変更](#コア網側_R3-R5_delayメトリックの変更)
   - [疎通確認_internet_delayメトリック変更後](#疎通確認_internet_delayメトリック変更後)
   - [疎通確認_gamer_delayメトリック変更後](#疎通確認_gamer_delayメトリック変更後)

# clab-sr-r1 バックアップ設定

<details>
<summary>階層化コンフィグ</summary>

```bash
    /configure card 1 card-type i24-800g-qsfpdd-1
    /configure card 1 level he2800g+
    /configure card 1 mda 1 mda-type m24-800g-qsfpdd-1
    /configure log filter "1001" named-entry "10" description "Collect only events of major severity or higher"
    /configure log filter "1001" named-entry "10" action forward
    /configure log filter "1001" named-entry "10" match severity gte major
    /configure log log-id "10" source main true
    /configure log log-id "10" source security true
    /configure log log-id "10" source change true
    /configure { log log-id "10" destination snmp }
    /configure log log-id "20" source main true
    /configure log log-id "20" source security true
    /configure log log-id "20" source change true
    /configure log log-id "20" destination syslog "1"
    /configure log log-id "30" source main true
    /configure log log-id "30" source security true
    /configure log log-id "30" source change true
    /configure { log log-id "30" destination cli }
    /configure log log-id "99" description "Default System Log"
    /configure log log-id "99" source main true
    /configure log log-id "99" destination memory max-entries 500
    /configure log log-id "100" description "Default Serious Errors Log"
    /configure log log-id "100" filter "1001"
    /configure log log-id "100" source main true
    /configure log log-id "100" destination memory max-entries 500
    /configure log snmp-trap-group "10" trap-target "snmptrapd" address 172.20.20.1
    /configure log snmp-trap-group "10" trap-target "snmptrapd" port 162
    /configure log snmp-trap-group "10" trap-target "snmptrapd" version snmpv2c
    /configure log snmp-trap-group "10" trap-target "snmptrapd" notify-community "public"
    /configure log syslog "1" address 172.20.20.1
    /configure log syslog "1" port 514
    /configure { policy-options community "customer1-export" member "target:65000:1" }
    /configure { policy-options community "customer1-import" member "target:65000:1" }
    /configure { policy-options community "customer10-export" member "target:65000:10" }
    /configure { policy-options community "customer10-import" member "target:65000:10" }
    /configure { policy-options prefix-list "gaming" prefix 11.0.1.0/24 type exact }
    /configure { policy-options prefix-list "gaming" prefix 11.0.2.0/24 type exact }
    /configure { policy-options prefix-list "internet" prefix 10.0.1.0/24 type exact }
    /configure { policy-options prefix-list "internet" prefix 10.0.2.0/24 type exact }
    /configure policy-options policy-statement "customer1-import" entry 10 from prefix-list ["internet"]
    /configure policy-options policy-statement "customer1-import" entry 10 from community name "customer1-import"
    /configure policy-options policy-statement "customer1-import" entry 10 action action-type accept
    /configure policy-options policy-statement "customer1-import" entry 20 from prefix-list ["gaming"]
    /configure policy-options policy-statement "customer1-import" entry 20 from community name "customer1-import"
    /configure policy-options policy-statement "customer1-import" entry 20 action action-type accept
    /configure policy-options policy-statement "customer1-import" entry 20 action flex-algo 128
    /configure policy-options policy-statement "customer1-import" default-action action-type accept
    /configure policy-options policy-statement "customer10-import" entry 10 from prefix-list ["internet"]
    /configure policy-options policy-statement "customer10-import" entry 10 from community name "customer10-import"
    /configure policy-options policy-statement "customer10-import" entry 10 action action-type accept
    /configure policy-options policy-statement "customer10-import" entry 20 from prefix-list ["gaming"]
    /configure policy-options policy-statement "customer10-import" entry 20 from community name "customer10-import"
    /configure policy-options policy-statement "customer10-import" entry 20 action action-type accept
    /configure policy-options policy-statement "customer10-import" entry 20 action flex-algo 128
    /configure policy-options policy-statement "customer10-import" default-action action-type accept
    /configure port 1/1/c1 admin-state enable
    /configure port 1/1/c1 connector breakout c1-100g
    /configure port 1/1/c1/1 admin-state enable
    /configure port 1/1/c1/1 ethernet mode hybrid
    /configure port 1/1/c1/1 ethernet mtu 9242
    /configure port 1/1/c2 admin-state enable
    /configure port 1/1/c2 connector breakout c1-100g
    /configure port 1/1/c2/1 admin-state enable
    /configure port 1/1/c2/1 ethernet mode hybrid
    /configure port 1/1/c2/1 ethernet mtu 9800
    /configure port 1/1/c3 admin-state enable
    /configure port 1/1/c3 connector breakout c1-100g
    /configure port 1/1/c3/1 admin-state enable
    /configure port 1/1/c3/1 ethernet mode hybrid
    /configure port 1/1/c3/1 ethernet mtu 9800
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
    /configure router "Base" mpls-labels sr-labels start 100000
    /configure router "Base" mpls-labels sr-labels end 100999
    /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp group "iBGP" peer-as 65000
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp group "iBGP" family evpn true
    /configure router "Base" bgp neighbor "192.0.2.3" group "iBGP"
    /configure router "Base" isis 0 admin-state enable
    /configure router "Base" isis 0 advertise-router-capability as
    /configure router "Base" isis 0 level-capability 2
    /configure router "Base" isis 0 traffic-engineering false
    /configure router "Base" isis 0 area-address [49.0001]
    /configure router "Base" isis 0 flexible-algorithms admin-state enable
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 participate true
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 advertise "Flex-Algo-128"
    /configure { router "Base" isis 0 traffic-engineering-options application-link-attributes }
    /configure router "Base" isis 0 segment-routing admin-state enable
    /configure router "Base" isis 0 segment-routing prefix-sid-range global
    /configure router "Base" isis 0 interface "system" ipv4-node-sid index 1
    /configure router "Base" isis 0 interface "system" flex-algo 128 ipv4-node-sid index 11
    /configure router "Base" isis 0 interface "to_R3" interface-type point-to-point
    /configure { router "Base" isis 0 interface "to_R3" level 1 }
    /configure router "Base" isis 0 interface "to_R4" interface-type point-to-point
    /configure { router "Base" isis 0 interface "to_R4" level 1 }
    /configure router "Base" isis 0 level 2 wide-metrics-only true
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" admin-state enable
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" description "Flex-Algo for Delay Metric"
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" metric-type delay
    /configure service customer "1" customer-id 1
    /configure service customer "10" description "L3-IP"
    /configure service customer "10" customer-id 10
    /configure service customer "10" contact "Nokia"
    /configure service customer "10" phone "+81-000-0000-0000"
    /configure service customer "20" description "L2-ELAN"
    /configure service customer "20" customer-id 20
    /configure service customer "20" contact "Nokia"
    /configure service customer "20" phone "+81-000-0000-0000"
    /configure service customer "30" description "L2-VPWS"
    /configure service customer "30" customer-id 30
    /configure service customer "30" contact "Nokia"
    /configure service customer "30" phone "+81-000-0000-0000"
    /configure service md-auto-id service-id-range start 1000
    /configure service md-auto-id service-id-range end 10000
    /configure service epipe "customer30" admin-state enable
    /configure service epipe "customer30" service-id 30
    /configure service epipe "customer30" customer "30"
    /configure service epipe "customer30" service-mtu 9228
    /configure service epipe "customer30" bgp 1 route-distinguisher "192.0.2.1:30"
    /configure service epipe "customer30" bgp 1 route-target export "target:65000:30"
    /configure service epipe "customer30" bgp 1 route-target import "target:65000:30"
    /configure service epipe "customer30" sap 1/1/c1/1:30 admin-state enable
    /configure service epipe "customer30" bgp-evpn evi 30
    /configure service epipe "customer30" bgp-evpn local-attachment-circuit "r1-ce" eth-tag 30
    /configure service epipe "customer30" bgp-evpn remote-attachment-circuit "r2-ce" eth-tag 30
    /configure service epipe "customer30" bgp-evpn mpls 1 admin-state enable
    /configure service epipe "customer30" bgp-evpn mpls 1 control-word true
    /configure service epipe "customer30" bgp-evpn mpls 1 ecmp 4
    /configure service epipe "customer30" bgp-evpn mpls 1 auto-bind-tunnel resolution any
    /configure service vpls "customer20" admin-state enable
    /configure service vpls "customer20" service-id 20
    /configure service vpls "customer20" customer "20"
    /configure service vpls "customer20" bgp 1 route-distinguisher "192.0.2.1:20"
    /configure service vpls "customer20" bgp 1 route-target export "target:65000:20"
    /configure service vpls "customer20" bgp 1 route-target import "target:65000:20"
    /configure service vpls "customer20" bgp-evpn evi 20
    /configure service vpls "customer20" bgp-evpn routes mac-ip advertise true
    /configure service vpls "customer20" bgp-evpn routes mac-ip unknown-mac true
    /configure service vpls "customer20" bgp-evpn mpls 1 admin-state enable
    /configure service vpls "customer20" bgp-evpn mpls 1 auto-bind-tunnel resolution any
    /configure { service vpls "customer20" sap 1/1/c1/1:20 }
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
    /configure { service vprn "customer1" bgp }
    /configure service vprn "customer10" admin-state enable
    /configure service vprn "customer10" service-id 10
    /configure service vprn "customer10" customer "10"
    /configure service vprn "customer10" bgp-evpn mpls 1 admin-state enable
    /configure service vprn "customer10" bgp-evpn mpls 1 route-distinguisher "192.0.2.1:10"
    /configure service vprn "customer10" bgp-evpn mpls 1 evi 10
    /configure service vprn "customer10" bgp-evpn mpls 1 vrf-import policy ["customer10-import"]
    /configure service vprn "customer10" bgp-evpn mpls 1 vrf-target import-community "target:65000:10"
    /configure service vprn "customer10" bgp-evpn mpls 1 vrf-target export-community "target:65000:10"
    /configure service vprn "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution filter
    /configure service vprn "customer10" bgp-evpn mpls 1 auto-bind-tunnel allow-flex-algo-fallback true
    /configure service vprn "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution-filter sr-isis true
    /configure service vprn "customer10" interface "to_gamer" ipv4 primary address 11.0.1.1
    /configure service vprn "customer10" interface "to_gamer" ipv4 primary prefix-length 24
    /configure { service vprn "customer10" interface "to_gamer" sap 1/1/c1/1:11 }
    /configure service vprn "customer10" interface "to_internet" ipv4 primary address 10.0.1.1
    /configure service vprn "customer10" interface "to_internet" ipv4 primary prefix-length 24
    /configure { service vprn "customer10" interface "to_internet" sap 1/1/c1/1:10 }
    /configure system name "r1"
    /configure system grpc admin-state enable
    /configure system grpc allow-unsecure-connection
    /configure system grpc gnmi auto-config-save true
    /configure system grpc rib-api admin-state enable
    /configure system management-interface configuration-save configuration-backups 5
    /configure system management-interface configuration-save incremental-saves false
    /configure system management-interface netconf auto-config-save true
    /configure system management-interface netconf listen admin-state enable
    /configure system management-interface snmp packet-size 9216
    /configure system management-interface snmp streaming admin-state enable
    /configure system bluetooth advertising-timeout 30
    /configure system login-control idle-timeout none
    /configure system login-control ssh inbound-max-sessions 10
    /configure system login-control telnet inbound-max-sessions 10
    /configure system security telnet-server true
    /configure system security ftp-server true
    /configure system security aaa local-profiles profile "administrative" default-action permit-all
    /configure system security aaa local-profiles profile "administrative" entry 10 match "configure system security"
    /configure system security aaa local-profiles profile "administrative" entry 10 action permit
    /configure system security aaa local-profiles profile "administrative" entry 20 match "show system security"
    /configure system security aaa local-profiles profile "administrative" entry 20 action permit
    /configure system security aaa local-profiles profile "administrative" entry 30 match "tools perform security"
    /configure system security aaa local-profiles profile "administrative" entry 30 action permit
    /configure system security aaa local-profiles profile "administrative" entry 40 match "tools dump security"
    /configure system security aaa local-profiles profile "administrative" entry 40 action permit
    /configure system security aaa local-profiles profile "administrative" entry 50 match "admin system security"
    /configure system security aaa local-profiles profile "administrative" entry 50 action permit
    /configure system security aaa local-profiles profile "administrative" entry 100 match "configure li"
    /configure system security aaa local-profiles profile "administrative" entry 100 action deny
    /configure system security aaa local-profiles profile "administrative" entry 110 match "show li"
    /configure system security aaa local-profiles profile "administrative" entry 110 action deny
    /configure system security aaa local-profiles profile "administrative" entry 111 match "clear li"
    /configure system security aaa local-profiles profile "administrative" entry 111 action deny
    /configure system security aaa local-profiles profile "administrative" entry 112 match "tools dump li"
    /configure system security aaa local-profiles profile "administrative" entry 112 action deny
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization action true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization cancel-commit true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization close-session true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization commit true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization copy-config true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization create-subscription true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization delete-config true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization discard-changes true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization edit-config true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get-config true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get-data true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization get-schema true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization kill-session true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization lock true
    /configure system security aaa local-profiles profile "administrative" netconf base-op-authorization validate true
    /configure system security aaa local-profiles profile "default" entry 10 match "exec"
    /configure system security aaa local-profiles profile "default" entry 10 action permit
    /configure system security aaa local-profiles profile "default" entry 20 match "exit"
    /configure system security aaa local-profiles profile "default" entry 20 action permit
    /configure system security aaa local-profiles profile "default" entry 30 match "help"
    /configure system security aaa local-profiles profile "default" entry 30 action permit
    /configure system security aaa local-profiles profile "default" entry 40 match "logout"
    /configure system security aaa local-profiles profile "default" entry 40 action permit
    /configure system security aaa local-profiles profile "default" entry 50 match "password"
    /configure system security aaa local-profiles profile "default" entry 50 action permit
    /configure system security aaa local-profiles profile "default" entry 60 match "show config"
    /configure system security aaa local-profiles profile "default" entry 60 action deny
    /configure system security aaa local-profiles profile "default" entry 65 match "show li"
    /configure system security aaa local-profiles profile "default" entry 65 action deny
    /configure system security aaa local-profiles profile "default" entry 66 match "clear li"
    /configure system security aaa local-profiles profile "default" entry 66 action deny
    /configure system security aaa local-profiles profile "default" entry 67 match "tools dump li"
    /configure system security aaa local-profiles profile "default" entry 67 action deny
    /configure system security aaa local-profiles profile "default" entry 68 match "state li"
    /configure system security aaa local-profiles profile "default" entry 68 action deny
    /configure system security aaa local-profiles profile "default" entry 70 match "show"
    /configure system security aaa local-profiles profile "default" entry 70 action permit
    /configure system security aaa local-profiles profile "default" entry 75 match "state"
    /configure system security aaa local-profiles profile "default" entry 75 action permit
    /configure system security aaa local-profiles profile "default" entry 80 match "enable-admin"
    /configure system security aaa local-profiles profile "default" entry 80 action permit
    /configure system security aaa local-profiles profile "default" entry 90 match "enable"
    /configure system security aaa local-profiles profile "default" entry 90 action permit
    /configure system security aaa local-profiles profile "default" entry 100 match "configure li"
    /configure system security aaa local-profiles profile "default" entry 100 action deny
    /configure system security snmp community "76HzdddhlPpRo1Vql+ZB5spLqccgYQ== hash2" access-permissions r
    /configure system security snmp community "76HzdddhlPpRo1Vql+ZB5spLqccgYQ== hash2" version v2c
    /configure system security ssh server-cipher-list-v2 cipher 190 name aes256-ctr
    /configure system security ssh server-cipher-list-v2 cipher 192 name aes192-ctr
    /configure system security ssh server-cipher-list-v2 cipher 194 name aes128-ctr
    /configure system security ssh server-cipher-list-v2 cipher 200 name aes128-cbc
    /configure system security ssh server-cipher-list-v2 cipher 205 name 3des-cbc
    /configure system security ssh server-cipher-list-v2 cipher 225 name aes192-cbc
    /configure system security ssh server-cipher-list-v2 cipher 230 name aes256-cbc
    /configure system security ssh client-cipher-list-v2 cipher 190 name aes256-ctr
    /configure system security ssh client-cipher-list-v2 cipher 192 name aes192-ctr
    /configure system security ssh client-cipher-list-v2 cipher 194 name aes128-ctr
    /configure system security ssh client-cipher-list-v2 cipher 200 name aes128-cbc
    /configure system security ssh client-cipher-list-v2 cipher 205 name 3des-cbc
    /configure system security ssh client-cipher-list-v2 cipher 225 name aes192-cbc
    /configure system security ssh client-cipher-list-v2 cipher 230 name aes256-cbc
    /configure system security ssh server-mac-list-v2 mac 200 name hmac-sha2-512
    /configure system security ssh server-mac-list-v2 mac 210 name hmac-sha2-256
    /configure system security ssh server-mac-list-v2 mac 215 name hmac-sha1
    /configure system security ssh server-mac-list-v2 mac 220 name hmac-sha1-96
    /configure system security ssh server-mac-list-v2 mac 225 name hmac-md5
    /configure system security ssh server-mac-list-v2 mac 240 name hmac-md5-96
    /configure system security ssh client-mac-list-v2 mac 200 name hmac-sha2-512
    /configure system security ssh client-mac-list-v2 mac 210 name hmac-sha2-256
    /configure system security ssh client-mac-list-v2 mac 215 name hmac-sha1
    /configure system security ssh client-mac-list-v2 mac 220 name hmac-sha1-96
    /configure system security ssh client-mac-list-v2 mac 225 name hmac-md5
    /configure system security ssh client-mac-list-v2 mac 240 name hmac-md5-96
    /configure system security user-params local-user user "admin" password "$2y$10$TQrZlpBDra86.qoexZUzQeBXDY1FcdDhGWdD9lLxMuFyPVSm0OGy6"
    /configure system security user-params local-user user "admin" restricted-to-home false
    /configure system security user-params local-user user "admin" access console true
    /configure system security user-params local-user user "admin" access ftp true
    /configure system security user-params local-user user "admin" access netconf true
    /configure system security user-params local-user user "admin" access grpc true
    /configure system security user-params local-user user "admin" console member ["administrative"]
    /configure system security user-params local-user user "admin2" password "$2y$10$Y2w976F.elVC92ttAuZMI.qVNPeXMqykrlczZaIStpY5Y6q/tcKou"
    /configure system security user-params local-user user "admin2" restricted-to-home false
    /configure system security user-params local-user user "admin2" access console true
    /configure system security user-params local-user user "admin2" access ftp true
    /configure system security user-params local-user user "admin2" access ssh-cli true
    /configure system security user-params local-user user "admin2" console member ["administrative"]
    /configure system security user-params local-user user "guest" password "$2y$10$giCgIdHFt1mUqhiUYP8Yg.dmogwuZms4w.Gad5A3.UXFwCWVhb5vW"
    /configure system security user-params local-user user "guest" restricted-to-home false
    /configure system security user-params local-user user "guest" access console true
    /configure system security user-params local-user user "guest" access ftp true
    /configure system security user-params local-user user "guest" access ssh-cli true
    /configure system security user-params local-user user "guest" console member ["administrative"]
    /configure system time zone non-standard name "jst"
    /configure system time zone non-standard offset "09:00"
    /configure system time ntp admin-state enable
    /configure system time ntp peer 172.20.20.1 router-instance "management" version 4
    /configure system time ntp peer 172.20.20.1 router-instance "management" prefer true
```

</details>

# 1. 初期設定

## ハードウェア

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
    configure {
        card 1 {
            card-type i24-800g-qsfpdd-1
            level he2800g+
            mda 1 {
                mda-type m24-800g-qsfpdd-1
            }
        }
    }
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure card 1 card-type i24-800g-qsfpdd-1
    /configure card 1 level he2800g+
    /configure card 1 mda 1 mda-type m24-800g-qsfpdd-1

```

</details>

### ・ 確認コマンド

```bash
show card
show card detail
show mda
show mda detail
```

## NTP

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    system {
        time {
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
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure system time zone non-standard name "jst"
    /configure system time zone non-standard offset "09:00"
    /configure system time ntp admin-state enable
    /configure system time ntp peer 172.20.20.1 router-instance "management" version 4
    /configure system time ntp peer 172.20.20.1 router-instance "management" prefer true
```

</details>

### ・ 確認コマンド

```bash
show time
show system ntp
show system ntp detail
show system ntp all
show system ntp peers
```

## SNMP Trap

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    log {
        log-id "10" {
            source {
                main true
                security true
                change true
            }
            destination {
                snmp {
                }
            }
        }
        snmp-trap-group "10" {
            trap-target "snmptrapd" {
                address 172.20.20.1
                port 162
                version snmpv2c
                notify-community "public"
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

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

</details>

### ・ 確認コマンド

```bash
show log log-id
show log log-id "10" detail
show log log-collector
```

```bash
admin@DL360-G10-006:~$ sudo tcpdump -n -i any udp port 162 or udp port 514
[sudo] password for admin: admin
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

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    log {
        log-id "20" {
            source {
                main true
                security true
                change true
            }
            destination {
                syslog "1"
            }
        }
        syslog "1" {
            address 172.20.20.1
            port 514
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure log log-id "20" source main true
    /configure log log-id "20" source security true
    /configure log log-id "20" source change true
    /configure log log-id "20" destination syslog "1"
    /configure log syslog "1" address 172.20.20.1
    /configure log syslog "1" port 514
```

</details>

### ・ 確認コマンド

```bash
show log log-id
show log log-id "20" detail
show log log-collector
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

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    system {
        security {
            user-params {
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
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure system security user-params local-user user "guest" password admin123
    /configure system security user-params local-user user "guest" restricted-to-home false
    /configure system security user-params local-user user "guest" access console true
    /configure system security user-params local-user user "guest" access ftp true
    /configure system security user-params local-user user "guest" access ssh-cli true
    /configure system security user-params local-user user "guest" console member ["administrative"]
```

</details>


### ・ 確認コマンド

```bash
show system security user
show system security user detail
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

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    system {
        login-control {
            idle-timeout none
            ssh {
                inbound-max-sessions 10
            }
            telnet {
                inbound-max-sessions 10
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure system login-control idle-timeout none
    /configure system login-control ssh inbound-max-sessions 10
    /configure system login-control telnet inbound-max-sessions 10

```

</details>

### ・ 確認コマンド

```bash
show system security user
show system security user detail
show system security management
```

## ターミナルロギング

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    log {
        log-id "30" {
            source {
                main true
                security true
                change true
            }
            destination {
                cli {
                }
            }
        }
    }
}
```
</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure log log-id "30" source main true
    /configure log log-id "30" source security true
    /configure log log-id "30" source change true
    /configure log log-id "30" destination { cli }

```

</details>

### ・ 確認コマンド

```bash
show log log-id
show log log-id "30" detail
show log log-collector
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

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    router "Base" {
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
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure router "Base" interface "system" ipv4 primary address 192.0.2.1
    /configure router "Base" interface "system" ipv4 primary prefix-length 32
```

</details>

| ホスト名 | system loopback address (IPv4) | system loopback address (IPv6) |
| --- | --- | --- |
| clab-sr-r1 | 192.0.2.1 /32 | 192:2::1 /128 |
| clab-sr-r2 | 192.0.2.2 /32 | 192:2::2 /128 |
| clab-sr-r3 | 192.0.2.3 /32 | 192:2::3 /128 |
| clab-sr-r4 | 192.0.2.4 /32 | 192:2::4 /128 |
| clab-sr-r5 | 192.0.2.5 /32 | 192:2::5 /128 |
| clab-sr-r6 | 192.0.2.6 /32 | 192:2::6 /128 |

### ・ 確認コマンド

```bash
show router "Base" interface
show router interface
show router interface detail
```

# 2. 応用編

## 物理ポート設定

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    port 1/1/c1 {
        admin-state enable
        connector {
            breakout c1-100g
        }
    }
    port 1/1/c1/1 {
        admin-state enable
        ethernet {
            mode hybrid
            mtu 9242
        }
    }
    port 1/1/c2 {
        admin-state enable
        connector {
            breakout c1-100g
        }
    }
    port 1/1/c2/1 {
        admin-state enable
        ethernet {
            mode hybrid
            mtu 9800
        }
    }
    port 1/1/c3 {
        admin-state enable
        connector {
            breakout c1-100g
        }
    }
    port 1/1/c3/1 {
        admin-state enable
        ethernet {
            mode hybrid
            mtu 9800
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

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
</details>

### ・ 確認コマンド

```bash
show port
show port detail
show port 1/1/c1
show port 1/1/c1 detail
show port 1/1/c1/1
show port 1/1/c1/1 detail
```

## コア網側インターフェース設定

---

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    router "Base" {
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
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
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

</details>


### ・ 確認コマンド

```bash
show router "Base" interface
show router interface
show router interface detail
```

```bash
(gl)[/configure log log-id "30"]
A:admin@r1# ping 192.168.13.1 router-instance "Base"
PING 192.168.13.1 56 data bytes
64 bytes from 192.168.13.1: icmp_seq=1 ttl=64 time=7.69ms.
64 bytes from 192.168.13.1: icmp_seq=2 ttl=64 time=7.14ms.
64 bytes from 192.168.13.1: icmp_seq=3 ttl=64 time=7.36ms.
64 bytes from 192.168.13.1: icmp_seq=4 ttl=64 time=6.87ms.
64 bytes from 192.168.13.1: icmp_seq=5 ttl=64 time=6.72ms.

---- 192.168.13.1 PING Statistics ----
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min = 6.72ms, avg = 7.16ms, max = 7.69ms, stddev = 0.345ms
```

## コア網側ISIS設定

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    router "Base" {
        isis 0 {
            admin-state enable
            advertise-router-capability as
            level-capability 2
            traffic-engineering false
            area-address [49.0001]
            interface "system" {
                level 1 {
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
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure router "Base" isis 0 admin-state enable
    /configure router "Base" isis 0 advertise-router-capability as
    /configure router "Base" isis 0 level-capability 2
    /configure router "Base" isis 0 traffic-engineering false
    /configure router "Base" isis 0 area-address [49.0001]
    /configure router "Base" isis 0 interface "system" { level 1}
    /configure router "Base" isis 0 interface "to_R3" interface-type point-to-point
    /configure router "Base" isis 0 interface "to_R3" { level 1 }
    /configure router "Base" isis 0 interface "to_R4" interface-type point-to-point
    /configure router "Base" isis 0 interface "to_R4" { level 1 }
    /configure router "Base" isis 0 level 2 wide-metrics-only true
```

</details>

### ・ 確認コマンド

```bash
show router isis status
show router isis interface
show router isis adjacency
show router isis database
show router isis database detail
show router isis statistics
show router isis spf-log
show router route-table
```

## コア網側ISIS-SR設定

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    mpls-labels {
        sr-labels {
            start 100000
            end 100999
        }
    }
    router "Base" {
        isis 0 {
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
        }
    }
    routing-options {
        flexible-algorithm-definitions {
            flex-algo "Flex-Algo-128" {
                admin-state enable
                description "Flex-Algo for Delay Metric"
                metric-type delay
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure router "Base" mpls-labels sr-labels start 100000
    /configure router "Base" mpls-labels sr-labels end 100999
    /configure router "Base" isis 0 flexible-algorithms admin-state enable
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 participate true
    /configure router "Base" isis 0 flexible-algorithms flex-algo 128 advertise "Flex-Algo-128"
    /configure router "Base" isis 0 { traffic-engineering-options application-link-attributes }
    /configure router "Base" isis 0 segment-routing admin-state enable
    /configure router "Base" isis 0 segment-routing prefix-sid-range global
    /configure router "Base" isis 0 interface "system" ipv4-node-sid index 1
    /configure router "Base" isis 0 interface "system" flex-algo 128 ipv4-node-sid index 11
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" admin-state enable
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" description "Flex-Algo for Delay Metric"
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" metric-type delay
```

</details>


### ・ 確認コマンド

```bash
show router isis sid-stats node
show router isis sid-stats node
show router isis sid-stats adj
show router isis flex-algo
```

## コア網側iBGP設定

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    router "Base" {
        autonomous-system 65000
        bgp {
            admin-state enable
            rapid-withdrawal true
            rapid-update {
                vpn-ipv4 true
                evpn true
            }
            group "iBGP" {
                peer-as 65000
                family {
                    vpn-ipv4 true
                    evpn true
                }
            }
            neighbor "192.0.2.3" {
                group "iBGP"
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp rapid-withdrawal true
    /configure router "Base" bgp rapid-update vpn-ipv4 true
    /configure router "Base" bgp rapid-update evpn true
    /configure router "Base" bgp group "iBGP" peer-as 65000
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp neighbor "192.0.2.3" group "iBGP"
```

</details>

### ・ 確認コマンド

```bash
show router bgp summary
show router bgp neighbor
show router bgp neighbor 192.0.2.3 received-routes evpn
show router bgp neighbor 192.0.2.3 advertised-routes evpn
```

## CE網側設定_カスタマー情報

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    service {
        customer "1" {
            customer-id 1
        }
        customer "10" {
            description "L3-IP"
            customer-id 10
            contact "Nokia"
            phone "+81-000-0000-0000"
        }
        customer "20" {
            description "L2-ELAN"
            customer-id 20
            contact "Nokia"
            phone "+81-000-0000-0000"
        }
        customer "30" {
            description "L2-VPWS"
            customer-id 30
            contact "Nokia"
            phone "+81-000-0000-0000"
        }
    }
}
```
</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure service customer "10" description "L3-IP"
    /configure service customer "10" customer-id 10
    /configure service customer "10" contact "Nokia"
    /configure service customer "10" phone "+81-000-0000-0000"
    /configure service customer "20" description "L2-ELAN"
    /configure service customer "20" customer-id 20
    /configure service customer "20" contact "Nokia"
    /configure service customer "20" phone "+81-000-0000-0000"
    /configure service customer "30" description "L2-VPWS"
    /configure service customer "30" customer-id 30
    /configure service customer "30" contact "Nokia"
    /configure service customer "30" phone "+81-000-0000-0000"
```
</details>

### ・ 確認コマンド

```bash
show service customer
```

## CE網側設定_EVPN_L3VPN

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    policy-options {
        community "customer10-export" {
            member "target:65000:10" { }
        }
        community "customer10-import" {
            member "target:65000:10" { }
        }
        prefix-list "gaming" {
            prefix 11.0.1.0/24 type exact {
            }
            prefix 11.0.2.0/24 type exact {
            }
        }
        prefix-list "internet" {
            prefix 10.0.1.0/24 type exact {
            }
            prefix 10.0.2.0/24 type exact {
            }
        }
        policy-statement "customer10-import" {
            entry 10 {
                from {
                    prefix-list ["internet"]
                    community {
                        name "customer10-import"
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
                        name "customer10-import"
                    }
                }
                action {
                    action-type accept
                    flex-algo 128
                }
            }
            default-action {
                action-type accept
            }
        }
    }
    service {
        customer "1" {
            customer-id 1
        }
        customer "10" {
            description "L3-IP"
            customer-id 10
            contact "Nokia"
            phone "+81-000-0000-0000"
        }
        vprn "customer10" {
            admin-state enable
            service-id 10
            customer "10"
            bgp-evpn {
                mpls 1 {
                    admin-state enable
                    route-distinguisher "192.0.2.1:10"
                    evi 10
                    vrf-import {
                        policy ["customer10-import"]
                    }
                    vrf-target {
                        import-community "target:65000:10"
                        export-community "target:65000:10"
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
            interface "to_gamer" {
                ipv4 {
                    primary {
                        address 11.0.1.1
                        prefix-length 24
                    }
                }
                sap 1/1/c1/1:11 {
                }
            }
            interface "to_internet" {
                ipv4 {
                    primary {
                        address 10.0.1.1
                        prefix-length 24
                    }
                }
                sap 1/1/c1/1:10 {
                }
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure policy-options community "customer10-export" { member "target:65000:10" }
    /configure policy-options community "customer10-import" { member "target:65000:10" }

    /configure policy-options prefix-list "gaming" { prefix 11.0.1.0/24 type exact }
    /configure policy-options prefix-list "gaming" { prefix 11.0.2.0/24 type exact }
    /configure policy-options prefix-list "internet" { prefix 10.0.1.0/24 type exact }
    /configure policy-options prefix-list "internet" { prefix 10.0.2.0/24 type exact }

    /configure policy-options policy-statement "customer10-import" entry 10 from prefix-list ["internet"]
    /configure policy-options policy-statement "customer10-import" entry 10 from community name "customer10-import"
    /configure policy-options policy-statement "customer10-import" entry 10 action action-type accept
    /configure policy-options policy-statement "customer10-import" entry 20 from prefix-list ["gaming"]
    /configure policy-options policy-statement "customer10-import" entry 20 from community name "customer10-import"
    /configure policy-options policy-statement "customer10-import" entry 20 action action-type accept
    /configure policy-options policy-statement "customer10-import" entry 20 action flex-algo 128
    /configure policy-options policy-statement "customer10-import" default-action action-type accept

    /configure service customer "10" description "L3-IP"
    /configure service customer "10" customer-id 10
    /configure service customer "10" contact "Nokia"
    /configure service customer "10" phone "+81-000-0000-0000"

    /configure service vprn "customer10" admin-state enable
    /configure service vprn "customer10" service-id 10
    /configure service vprn "customer10" customer "10"
    /configure service vprn "customer10" bgp-evpn mpls 1 admin-state enable
    /configure service vprn "customer10" bgp-evpn mpls 1 route-distinguisher "192.0.2.1:10"
    /configure service vprn "customer10" bgp-evpn mpls 1 evi 10
    /configure service vprn "customer10" bgp-evpn mpls 1 vrf-import policy ["customer10-import"]
    /configure service vprn "customer10" bgp-evpn mpls 1 vrf-target import-community "target:65000:10"
    /configure service vprn "customer10" bgp-evpn mpls 1 vrf-target export-community "target:65000:10"
    /configure service vprn "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution filter
    /configure service vprn "customer10" bgp-evpn mpls 1 auto-bind-tunnel allow-flex-algo-fallback true
    /configure service vprn "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution-filter sr-isis true
    /configure service vprn "customer10" interface "to_gamer" ipv4 primary address 11.0.1.1
    /configure service vprn "customer10" interface "to_gamer" ipv4 primary prefix-length 24
    /configure service vprn "customer10" interface "to_gamer" { sap 1/1/c1/1:11 }
    /configure service vprn "customer10" interface "to_internet" ipv4 primary address 10.0.1.1
    /configure service vprn "customer10" interface "to_internet" ipv4 primary prefix-length 24
    /configure service vprn "customer10" interface "to_internet" { sap 1/1/c1/1:10 }
```

</details>

### ・ 確認コマンド

```bash
show router 10 interface
show router 10 route-table
show router bgp neighbor 192.0.2.3 received-routes evpn ip-prefix
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 ping -c 3 10.0.2.10
PING 10.0.2.10 (10.0.2.10) 56(84) bytes of data.
64 bytes from 10.0.2.10: icmp_seq=1 ttl=62 time=14.6 ms
64 bytes from 10.0.2.10: icmp_seq=2 ttl=62 time=15.3 ms
64 bytes from 10.0.2.10: icmp_seq=3 ttl=62 time=12.9 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 ping -c 3 11.0.2.10
PING 11.0.2.10 (11.0.2.10) 56(84) bytes of data.
64 bytes from 11.0.2.10: icmp_seq=1 ttl=62 time=165 ms
64 bytes from 11.0.2.10: icmp_seq=2 ttl=62 time=216 ms
64 bytes from 11.0.2.10: icmp_seq=3 ttl=62 time=258 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 traceroute 10.0.2.10
traceroute to 10.0.2.10 (10.0.2.10), 30 hops max, 46 byte packets
 1  10.0.1.1 (10.0.1.1)  177.461 ms  4.123 ms  4.496 ms
 2  10.0.2.1 (10.0.2.1)  17.409 ms  15.089 ms  12.812 ms
 3  10.0.2.10 (10.0.2.10)  11.940 ms  12.648 ms  13.295 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 traceroute 11.0.2.10
traceroute to 11.0.2.10 (11.0.2.10), 30 hops max, 46 byte packets
 1  11.0.1.1 (11.0.1.1)  119.219 ms  3.620 ms  4.306 ms
 2  11.0.2.1 (11.0.2.1)  14.379 ms  14.743 ms  13.775 ms
 3  11.0.2.10 (11.0.2.10)  13.289 ms  12.567 ms  12.647 ms
```

## CE網側設定_EVPN_L2VPN_ELAN

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    service {
        customer "20" {
            description "L2-ELAN"
            customer-id 20
            contact "Nokia"
            phone "+81-000-0000-0000"
        }
        vpls "customer20" {
            admin-state enable
            service-id 20
            customer "20"
            bgp 1 {
                route-distinguisher "192.0.2.1:20"
                route-target {
                    export "target:65000:20"
                    import "target:65000:20"
                }
            }
            bgp-evpn {
                evi 20
                routes {
                    mac-ip {
                        advertise true
                        unknown-mac true
                    }
                }
                mpls 1 {
                    admin-state enable
                    auto-bind-tunnel {
                        resolution any
                    }
                }
            }
            sap 1/1/c1/1:20 {
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure service vpls "customer20" admin-state enable
    /configure service vpls "customer20" service-id 20
    /configure service vpls "customer20" customer "20"
    /configure service vpls "customer20" bgp 1 route-distinguisher "192.0.2.1:20"
    /configure service vpls "customer20" bgp 1 route-target export "target:65000:20"
    /configure service vpls "customer20" bgp 1 route-target import "target:65000:20"
    /configure service vpls "customer20" bgp-evpn evi 20
    /configure service vpls "customer20" bgp-evpn routes mac-ip advertise true
    /configure service vpls "customer20" bgp-evpn routes mac-ip unknown-mac true
    /configure service vpls "customer20" bgp-evpn mpls 1 admin-state enable
    /configure service vpls "customer20" bgp-evpn mpls 1 auto-bind-tunnel resolution any
    /configure service vpls "customer20" sap 1/1/c1/1:20 { }
```

</details>

### ・ 確認コマンド

```bash
show router bgp neighbor 192.0.2.3 received-routes evpn mac
show router bgp neighbor 192.0.2.3 advertised-routes evpn mac
show service id "customer20" all
show service id "customer20" fdb
show service id "customer20" fdb detail
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 ping -c 3 20.0.0.2
PING 20.0.0.2 (20.0.0.2) 56(84) bytes of data.
64 bytes from 20.0.0.2: icmp_seq=1 ttl=64 time=14.0 ms
64 bytes from 20.0.0.2: icmp_seq=2 ttl=64 time=13.8 ms
64 bytes from 20.0.0.2: icmp_seq=3 ttl=64 time=14.8 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 ping -c 3 20.0.0.6
PING 20.0.0.6 (20.0.0.6) 56(84) bytes of data.
64 bytes from 20.0.0.6: icmp_seq=1 ttl=64 time=32.5 ms
64 bytes from 20.0.0.6: icmp_seq=2 ttl=64 time=79.6 ms
64 bytes from 20.0.0.6: icmp_seq=3 ttl=64 time=133 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 traceroute 20.0.0.2
traceroute to 20.0.0.2 (20.0.0.2), 30 hops max, 46 byte packets
 1  20.0.0.2 (20.0.0.2)  323.570 ms  15.303 ms  14.068 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 traceroute 20.0.0.6
traceroute to 20.0.0.6 (20.0.0.6), 30 hops max, 46 byte packets
 1  20.0.0.6 (20.0.0.6)  92.586 ms  9.591 ms  10.061 ms
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 traceroute 20.0.0.6
traceroute to 20.0.0.6 (20.0.0.6), 30 hops max, 46 byte packets
 1  20.0.0.6 (20.0.0.6)  92.586 ms  9.591 ms  10.061 ms
```


## CE網側設定_EVPN_L2VPN_VPWS

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

```bash
configure {
    service {
        customer "30" {
            description "L2-VPWS"
            customer-id 30
            contact "Nokia"
            phone "+81-000-0000-0000"
        }
        epipe "customer30" {
            admin-state enable
            service-id 30
            customer "30"
            service-mtu 9228
            bgp 1 {
                route-distinguisher "192.0.2.1:30"
                route-target {
                    export "target:65000:30"
                    import "target:65000:30"
                }
            }
            sap 1/1/c1/1:30 {
                admin-state enable
            }
            bgp-evpn {
                evi 30
                local-attachment-circuit "r1-ce" {
                    eth-tag 30
                }
                remote-attachment-circuit "r2-ce" {
                    eth-tag 30
                }
                mpls 1 {
                    admin-state enable
                    control-word true
                    ecmp 4
                    auto-bind-tunnel {
                        resolution any
                    }
                }
            }
        }
    }
}
```

</details>

<details>
<summary>フラットコンフィグ</summary>

```bash
    /configure service epipe "customer30" admin-state enable
    /configure service epipe "customer30" service-id 30
    /configure service epipe "customer30" customer "30"
    /configure service epipe "customer30" service-mtu 9228
    /configure service epipe "customer30" bgp 1 route-distinguisher "192.0.2.1:30"
    /configure service epipe "customer30" bgp 1 route-target export "target:65000:30"
    /configure service epipe "customer30" bgp 1 route-target import "target:65000:30"
    /configure service epipe "customer30" sap 1/1/c1/1:30 admin-state enable
    /configure service epipe "customer30" bgp-evpn evi 30
    /configure service epipe "customer30" bgp-evpn local-attachment-circuit "r1-ce" eth-tag 30
    /configure service epipe "customer30" bgp-evpn remote-attachment-circuit "r2-ce" eth-tag 30
    /configure service epipe "customer30" bgp-evpn mpls 1 admin-state enable
    /configure service epipe "customer30" bgp-evpn mpls 1 control-word true
    /configure service epipe "customer30" bgp-evpn mpls 1 ecmp 4
    /configure service epipe "customer30" bgp-evpn mpls 1 auto-bind-tunnel resolution any
```

</details>

### ・ 確認コマンド

```bash
show router bgp neighbor 192.0.2.3 received-routes evpn auto-disc
show router bgp neighbor 192.0.2.3 advertised-routes evpn auto-disc
show service id "customer30" all
```

```bash
root@pod4-VM:~# docker exec -it clab-sr-host1 ping -c 3 20.0.0.2
PING 20.0.0.2 (20.0.0.2) 56(84) bytes of data.
64 bytes from 20.0.0.2: icmp_seq=1 ttl=64 time=14.0 ms
64 bytes from 20.0.0.2: icmp_seq=2 ttl=64 time=13.8 ms
64 bytes from 20.0.0.2: icmp_seq=3 ttl=64 time=14.8 ms
```

## 疎通確認_internet_delayメトリック変更前

### ・ 確認コマンド

```bash
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh start internet
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
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh stop internet
Stopping traffic

```

## 疎通確認_gamer_delayメトリック変更前

### ・ 確認コマンド

```bash
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh start gamer
Starting gamer traffic
Connecting to host 11.0.2.10, port 5202

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
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh stop gamer
Stopping traffic

```

## コア網側_R3-R5_delayメトリックの変更

### ・ 設定変更

<details>
<summary>階層化コンフィグ</summary>

R3 ターミナル
```bash
configure {
    router "Base" {
        interface "to_R5" {
            if-attribute {
                delay {
                    static 50000
                }
            }
        }
    }
}
```

</details>

### ・ 確認コマンド

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

## 疎通確認_internet_delayメトリック変更後

### ・ 確認コマンド


```bash
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh start internet
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
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh stop internet
Stopping traffic

```

## 疎通確認_gamer_delayメトリック変更後

---

### ・ 確認コマンド

```bash
root@pod1-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh start gamer
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
root@pod5-KVM:/home/clab/sros-hands-on# sudo /home/clab/pod1/traffic.sh stop gamer
Stopping traffic

```
