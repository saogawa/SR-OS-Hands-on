## 目次

1. [初期設定](#1-初期設定)
   - [ハードウェア](#ハードウェア)
   - [NTP](#ntp)
   - [SNMP Trap](#snmp-trap)
   - [Syslog](#syslog)
   - [ユーザアカウント](#ユーザアカウント)
   - [ログインコントロール](#ログインコントロール)
   - [ターミナルロギング](#ターミナルロギング)
   - [systemループバック](#systemループバック)
2. [応用編](#2-応用編)
   - [物理ポート設定](#物理ポート設定)
   - [コア網側インターフェース設定](#コア網側インターフェース設定)
   - [コア網側OSPFv2設定](#コア網側ospfv2設定)
   - [コア網側SR-MPLS設定](#コア網側sr-mpls設定)
   - [コア網側iBGP設定](#コア網側ibgp設定)
   - [CE網側設定_カスタマー情報](#ce網側設定_カスタマー情報)
   - [CE網側設定_EVPN_L2VPN_ELAN](#ce網側設定_evpn_l2vpn_elan)


# clab-sr-r1 バックアップ設定

<details>
<summary>R1</summary>

```bash
    /configure card 1 card-type iom-1
    /configure card 1 mda 1 mda-type me12-100gb-qsfp28
    /configure log filter "1001" named-entry "10" description "Collect only events of major severity or higher"
    /configure log filter "1001" named-entry "10" action forward
    /configure log filter "1001" named-entry "10" match severity gte major
    /configure log log-id "99" description "Default System Log"
    /configure log log-id "99" source main true
    /configure log log-id "99" destination memory max-entries 500
    /configure log log-id "100" description "Default Serious Errors Log"
    /configure log log-id "100" filter "1001"
    /configure log log-id "100" source main true
    /configure log log-id "100" destination memory max-entries 500
    /configure policy-options policy-statement "customer10-export" default-action action-type accept
    /configure policy-options policy-statement "customer10-import" default-action action-type accept
    /configure port 1/1/c1 admin-state enable
    /configure port 1/1/c1 connector breakout c1-100g
    /configure port 1/1/c1/1 admin-state enable
    /configure port 1/1/c1/1 ethernet mode hybrid
    /configure port 1/1/c1/1 ethernet mtu 9800
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
    /configure router "Base" bgp rapid-withdrawal true
    /configure router "Base" bgp rapid-update vpn-ipv4 true
    /configure router "Base" bgp rapid-update evpn true
    /configure router "Base" bgp group "iBGP" peer-as 65000
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp group "iBGP" family evpn true
    /configure router "Base" bgp neighbor "192.0.2.3" group "iBGP"
    /configure router "Base" ospf 0 admin-state enable
    /configure router "Base" ospf 0 router-id 192.168.2.1
    /configure router "Base" ospf 0 advertise-router-capability area
    /configure router "Base" ospf 0 traffic-engineering false
    /configure router "Base" ospf 0 segment-routing admin-state enable
    /configure router "Base" ospf 0 segment-routing entropy-label true
    /configure router "Base" ospf 0 segment-routing prefix-sid-range global
    /configure router "Base" ospf 0 segment-routing egress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing egress-statistics node-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics node-sid true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" passive true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" node-sid index 1
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R3" interface-type point-to-point
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R4" interface-type point-to-point
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" admin-state enable
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" description "Flex-Algo for Delay Metric"
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" metric-type delay
    /configure service customer "10" description "L2-ELAN"
    /configure service customer "10" customer-id 10
    /configure service customer "10" contact "Nokia"
    /configure service customer "10" phone "+81-000-0000-0000"
    /configure service vpls "customer10" admin-state enable
    /configure service vpls "customer10" service-id 10
    /configure service vpls "customer10" customer "10"
    /configure service vpls "customer10" bgp 1 route-distinguisher "65000:10"
    /configure service vpls "customer10" bgp 1 route-target export "target:65000:10"
    /configure service vpls "customer10" bgp 1 route-target import "target:65000:10"
    /configure service vpls "customer10" bgp-evpn evi 10
    /configure service vpls "customer10" bgp-evpn mpls 1 admin-state enable
    /configure service vpls "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution filter
    /configure service vpls "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution-filter sr-ospf true
    /configure service { vpls "customer10" sap 1/1/c1/1:10 }
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
    /configure system login-control ssh inbound-max-sessions 30
    /configure system security aaa local-profiles profile "administrative" default-action permit-all
    /configure system security aaa local-profiles profile "administrative" entry 10 match "configure system security"
    /configure system security aaa local-profiles profile "administrative" entry 10 action permit
    /configure system security aaa local-profiles profile "administrative" entry 20 match "show system security"
    /configure system security aaa local-profiles profile "administrative" entry 20 action permit
    /configure system security aaa local-profiles profile "administrative" entry 30 match "tools perform security"
    /configure system security aaa local-profiles profile "administrative" entry 30 action permit
    /configure system security aaa local-profiles profile "administrative" entry 40 match "tools dump security"
    /configure system security aaa local-profiles profile "administrative" entry 40 action permit
    /configure system security aaa local-profiles profile "administrative" entry 42 match "tools dump system security"
    /configure system security aaa local-profiles profile "administrative" entry 42 action permit
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
```

</details>

<details>
<summary>R2</summary>

```bash
    /configure card 1 card-type iom-1
    /configure card 1 mda 1 mda-type me12-100gb-qsfp28
    /configure log filter "1001" named-entry "10" description "Collect only events of major severity or higher"
    /configure log filter "1001" named-entry "10" action forward
    /configure log filter "1001" named-entry "10" match severity gte major
    /configure log log-id "99" description "Default System Log"
    /configure log log-id "99" source main true
    /configure log log-id "99" destination memory max-entries 500
    /configure log log-id "100" description "Default Serious Errors Log"
    /configure log log-id "100" filter "1001"
    /configure log log-id "100" source main true
    /configure log log-id "100" destination memory max-entries 500
    /configure policy-options { community "customer1-export" member "target:65000:1" }
    /configure policy-options { community "customer1-import" member "target:65000:1" }
    /configure policy-options { prefix-list "gaming" prefix 20.0.1.0/24 type exact }
    /configure policy-options { prefix-list "internet" prefix 10.0.1.0/24 type exact }
    /configure policy-options policy-statement "customer1-import" entry 10 from prefix-list ["internet"]
    /configure policy-options policy-statement "customer1-import" entry 10 from community name "customer1-import"
    /configure policy-options policy-statement "customer1-import" entry 10 action action-type accept
    /configure policy-options policy-statement "customer1-import" entry 20 from prefix-list ["gaming"]
    /configure policy-options policy-statement "customer1-import" entry 20 from community name "customer1-import"
    /configure policy-options policy-statement "customer1-import" entry 20 action action-type accept
    /configure policy-options policy-statement "customer1-import" entry 20 action flex-algo 128
    /configure policy-options policy-statement "customer1-import" default-action action-type reject
    /configure port 1/1/c1 admin-state enable
    /configure port 1/1/c1 connector breakout c1-100g
    /configure port 1/1/c1/1 admin-state enable
    /configure port 1/1/c1/1 ethernet mode hybrid
    /configure port 1/1/c1/1 ethernet mtu 9800
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
    /configure router "Base" interface "system" ipv4 primary address 192.0.2.2
    /configure router "Base" interface "system" ipv4 primary prefix-length 32
    /configure router "Base" interface "to_R5" admin-state enable
    /configure router "Base" interface "to_R5" port 1/1/c2/1:0
    /configure router "Base" interface "to_R5" ipv4 primary address 192.168.25.0
    /configure router "Base" interface "to_R5" ipv4 primary prefix-length 31
    /configure router "Base" interface "to_R5" if-attribute delay static 10000
    /configure router "Base" interface "to_R6" admin-state enable
    /configure router "Base" interface "to_R6" port 1/1/c3/1:0
    /configure router "Base" interface "to_R6" ipv4 primary address 192.168.26.0
    /configure router "Base" interface "to_R6" ipv4 primary prefix-length 31
    /configure router "Base" interface "to_R6" if-attribute delay static 10000
    /configure router "Base" mpls-labels sr-labels start 100000
    /configure router "Base" mpls-labels sr-labels end 100999
    /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp group "iBGP" peer-as 65000
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp group "iBGP" family evpn true
    /configure router "Base" bgp neighbor "192.0.2.3" group "iBGP"
    /configure router "Base" ospf 0 admin-state enable
    /configure router "Base" ospf 0 router-id 192.168.2.2
    /configure router "Base" ospf 0 advertise-router-capability area
    /configure router "Base" ospf 0 traffic-engineering false
    /configure router "Base" ospf 0 segment-routing admin-state enable
    /configure router "Base" ospf 0 segment-routing entropy-label true
    /configure router "Base" ospf 0 segment-routing prefix-sid-range global
    /configure router "Base" ospf 0 segment-routing egress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing egress-statistics node-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics node-sid true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" passive true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" node-sid index 2
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R5" interface-type point-to-point
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R6" interface-type point-to-point
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" admin-state enable
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" description "Flex-Algo for Delay Metric"
    /configure routing-options flexible-algorithm-definitions flex-algo "Flex-Algo-128" metric-type delay
    /configure service customer "10" description "L2-ELAN"
    /configure service customer "10" customer-id 10
    /configure service customer "10" contact "Nokia"
    /configure service customer "10" phone "+81-000-0000-0000"
    /configure service vpls "customer10" admin-state enable
    /configure service vpls "customer10" service-id 10
    /configure service vpls "customer10" customer "10"
    /configure service vpls "customer10" bgp 1 route-distinguisher "65000:10"
    /configure service vpls "customer10" bgp 1 route-target export "target:65000:10"
    /configure service vpls "customer10" bgp 1 route-target import "target:65000:10"
    /configure service vpls "customer10" bgp-evpn evi 10
    /configure service vpls "customer10" bgp-evpn routes mac-ip advertise true
    /configure service vpls "customer10" bgp-evpn routes mac-ip unknown-mac true
    /configure service vpls "customer10" bgp-evpn mpls 1 admin-state enable
    /configure service vpls "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution filter
    /configure service vpls "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution-filter sr-ospf true
    /configure service { vpls "customer10" sap 1/1/c1/1:10 }
    /configure system name "r2"
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
    /configure system login-control ssh inbound-max-sessions 30
    /configure system security aaa local-profiles profile "administrative" default-action permit-all
    /configure system security aaa local-profiles profile "administrative" entry 10 match "configure system security"
    /configure system security aaa local-profiles profile "administrative" entry 10 action permit
    /configure system security aaa local-profiles profile "administrative" entry 20 match "show system security"
    /configure system security aaa local-profiles profile "administrative" entry 20 action permit
    /configure system security aaa local-profiles profile "administrative" entry 30 match "tools perform security"
    /configure system security aaa local-profiles profile "administrative" entry 30 action permit
    /configure system security aaa local-profiles profile "administrative" entry 40 match "tools dump security"
    /configure system security aaa local-profiles profile "administrative" entry 40 action permit
    /configure system security aaa local-profiles profile "administrative" entry 42 match "tools dump system security"
    /configure system security aaa local-profiles profile "administrative" entry 42 action permit
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
```

</details>

# 1. 初期設定

## ハードウェア

### <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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

### <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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

### <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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
HOST-OS# sudo tcpdump -n -i any udp port 162 or udp port 514
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

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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
HOST-OS# sudo tcpdump -n -i any udp port 162 or udp port 514
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

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>コンフィグ</summary>

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
<summary>R1コンフィグ</summary>

```bash
    /configure router "Base" interface "system" ipv4 primary address 192.0.2.1
    /configure router "Base" interface "system" ipv4 primary prefix-length 32
```

</details>

<details>
<summary>R2コンフィグ</summary>

```bash
    /configure router "Base" interface "system" ipv4 primary address 192.0.2.2
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

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

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
<summary>R1 コンフィグ</summary>

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

<details>
<summary>R2 コンフィグ</summary>

```bash
    /configure router "Base" interface "to_R5" admin-state enable
    /configure router "Base" interface "to_R5" port 1/1/c2/1:0
    /configure router "Base" interface "to_R5" ipv4 primary address 192.168.25.0
    /configure router "Base" interface "to_R5" ipv4 primary prefix-length 31
    /configure router "Base" interface "to_R5" if-attribute delay static 10000
    /configure router "Base" interface "to_R6" admin-state enable
    /configure router "Base" interface "to_R6" port 1/1/c3/1:0
    /configure router "Base" interface "to_R6" ipv4 primary address 192.168.26.0
    /configure router "Base" interface "to_R6" ipv4 primary prefix-length 31
    /configure router "Base" interface "to_R6" if-attribute delay static 10000
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

## コア網側OSPFv2設定

### ・ 設定変更

<details>
<summary>R1コンフィグ</summary>

```bash
    /configure router "Base" ospf 0 admin-state enable
    /configure router "Base" ospf 0 router-id 192.168.2.1
    /configure router "Base" ospf 0 advertise-router-capability area
    /configure router "Base" ospf 0 traffic-engineering false
    /configure router "Base" ospf 0 segment-routing admin-state enable
    /configure router "Base" ospf 0 segment-routing entropy-label true
    /configure router "Base" ospf 0 segment-routing prefix-sid-range global
    /configure router "Base" ospf 0 segment-routing egress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing egress-statistics node-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics node-sid true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" passive true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" node-sid index 1
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R3" interface-type point-to-point
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R4" interface-type point-to-point
```

</details>

<details>
<summary>R2コンフィグ</summary>

```bash
    /configure router "Base" ospf 0 admin-state enable
    /configure router "Base" ospf 0 router-id 192.168.2.2
    /configure router "Base" ospf 0 advertise-router-capability area
    /configure router "Base" ospf 0 traffic-engineering falsea
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" passive true
    /configure router "Base" ospf 0 area 0.0.0.0 interface "system" node-sid index 2
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R5" interface-type point-to-point
    /configure router "Base" ospf 0 area 0.0.0.0 interface "to_R6" interface-type point-to-point
```

</details>

### ・ 確認コマンド

```bash
show router ospf status
show router ospf interface
show router ospf neighbor
show router ospf database
show router ospf database detail
show router ospf statistics
show router route-table
```

## コア網側SR-MPLS設定

### ・ 設定変更

<details>
<summary>R1コンフィグ</summary>

```bash
    /configure router "Base" mpls-labels sr-labels start 100000
    /configure router "Base" mpls-labels sr-labels end 100999
    /configure router "Base" ospf 0 segment-routing admin-state enable
    /configure router "Base" ospf 0 segment-routing entropy-label true
    /configure router "Base" ospf 0 segment-routing prefix-sid-range global
    /configure router "Base" ospf 0 segment-routing egress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing egress-statistics node-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics node-sid true
```

</details>

<details>
<summary>R2コンフィグ</summary>

```bash
    /configure router "Base" mpls-labels sr-labels start 100000
    /configure router "Base" mpls-labels sr-labels end 100999
    /configure router "Base" ospf 0 segment-routing admin-state enable
    /configure router "Base" ospf 0 segment-routing entropy-label true
    /configure router "Base" ospf 0 segment-routing prefix-sid-range global
    /configure router "Base" ospf 0 segment-routing egress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing egress-statistics node-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics adj-sid true
    /configure router "Base" ospf 0 segment-routing ingress-statistics node-sid true
```

</details>

### ・ 確認コマンド

```bash
show router ospf sid-stats summary
show router ospf sid-stats node
show router ospf opaque-database 
show router tunnel-table ipv4
tools dump router ospf 0 sr-adjacencies 
tools dump router segment-routing tunnel 
```

## コア網側iBGP設定

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更


<details>
<summary>R1/R2コンフィグ</summary>

```bash
    /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp rapid-withdrawal true
    /configure router "Base" bgp rapid-update vpn-ipv4 true
    /configure router "Base" bgp rapid-update evpn true
    /configure router "Base" bgp group "iBGP" peer-as 65000
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp group "iBGP" family evpn true
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

## <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1/R2コンフィグ</summary>

```bash
    /configure service customer "10" description "L2-ELAN"
    /configure service customer "10" customer-id 10
    /configure service customer "10" contact "Nokia"
    /configure service customer "10" phone "+81-000-0000-0000"
```
</details>

### ・ 確認コマンド

```bash
show service customer
```

## CE網側設定_EVPN_L2VPN_ELAN

#### <span style="color:blue">R1/R2共通</span>

### ・ 設定変更

<details>
<summary>R1コンフィグ</summary>

```bash
    /configure service vpls "customer10" admin-state enable
    /configure service vpls "customer10" service-id 10
    /configure service vpls "customer10" customer "10"
    /configure service vpls "customer10" bgp 1 route-distinguisher "65000:10"
    /configure service vpls "customer10" bgp 1 route-target export "target:65000:10"
    /configure service vpls "customer10" bgp 1 route-target import "target:65000:10"
    /configure service vpls "customer10" bgp-evpn evi 10
    /configure service vpls "customer10" bgp-evpn mpls 1 admin-state enable
    /configure service vpls "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution filter
    /configure service vpls "customer10" bgp-evpn mpls 1 auto-bind-tunnel resolution-filter sr-ospf true
    /configure service { vpls "customer10" sap 1/1/c1/1:10 }
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

#### <span style="color:blue">Services → HOME</span>

```bash
HOST-OS# docker exec -it clab-pod2-home ping 10.0.1.2
PING 10.0.1.2 (10.0.1.2) 56(84) bytes of data.
64 bytes from 10.0.1.2: icmp_seq=1 ttl=64 time=7.91 ms
64 bytes from 10.0.1.2: icmp_seq=2 ttl=64 time=7.42 ms
64 bytes from 10.0.1.2: icmp_seq=3 ttl=64 time=8.17 ms

HOST-OS# docker exec -it clab-pod2-home arp -an
? (10.0.1.2) at aa:c1:ab:71:a2:37 [ether] on eth1.10
```


#### <span style="color:blue">HOME → Services</span>

```bash
HOST-OS# docker exec -it clab-pod2-services ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 56(84) bytes of data.
64 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=8.07 ms
64 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=9.58 ms
64 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=7.94 ms

HOST-OS# docker exec -it clab-pod2-services arp -an
? (10.0.1.1) at aa:c1:ab:b2:39:39 [ether] on eth1.10

```

## 疎通確認

### ・ 確認コマンド

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
