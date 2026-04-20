## SR OS トレーニング手順書：EVPN-VXLAN (L2VPN) 構築

### ネットワーク構成イメージ

本構成は、アンダーレイに IPv6 BGP を使用し、オーバーレイで EVPN-VXLAN を構成することで、拠点間（CE間）のレイヤ2延伸を実現するアーキテクチャです。

-----

### 目次

1.  [システム基本設定](https://www.google.com/search?q=%231-%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E5%9F%BA%E6%9C%AC%E8%A8%AD%E5%AE%9A)
2.  [ロギング・通知設定](https://www.google.com/search?q=%232-%E3%83%AD%E3%82%AE%E3%83%B3%E3%82%B0%E3%83%BB%E9%80%9A%E7%9F%A5%E8%A8%AD%E5%AE%9A)
3.  [物理ポート・カード設定](https://www.google.com/search?q=%233-%E7%89%A9%E7%90%86%E3%83%9D%E3%83%BC%E3%83%88%E3%83%BB%E3%82%AB%E3%83%BC%E3%83%89%E8%A8%AD%E5%AE%9A)
4.  [LLDP 設定](https://www.google.com/search?q=%234-lldp-%E8%A8%AD%E5%AE%9A)
5.  [L3 ネットワーク設定 (Base Router)](https://www.google.com/search?q=%235-l3-%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E8%A8%AD%E5%AE%9A-base-router)
6.  [BFD 設定](https://www.google.com/search?q=%236-bfd-%E8%A8%AD%E5%AE%9A)
7.  [経路ポリシー設定](https://www.google.com/search?q=%237-%E7%B5%8C%E8%B7%AF%E3%83%9D%E3%83%AA%E3%82%B7%E3%83%BC%E8%A8%AD%E5%AE%9A)
8.  [BGP 設定 (Global/Underlay/Overlay)](https://www.google.com/search?q=%238-bgp-%E8%A8%AD%E5%AE%9A-globalunderlayoverlay)
9.  [L2VPN サービス設定 (VPLS)](https://www.google.com/search?q=%239-l2vpn-%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E8%A8%AD%E5%AE%9A-vpls)
10. [最終疎通確認](https://www.google.com/search?q=%2310-%E6%9C%80%E7%B5%82%E7%96%8E%E9%80%9A%E7%A2%BA%E8%AA%8D)

-----

<img width="1487" height="708" alt="Cursor_と_pod01_—_Hands-on__SSH__5_" src="https://github.com/user-attachments/assets/7029963d-323d-4076-8b84-d79c791d07af" />

#### ログイン方法

```bash
ssh clab-pod01-PE1 -l admin   ##   admin/admin
logout
```
```bash
❯ docker exec -it clab-pod01-CE1 sh
CE1 / # 
CE1 / # 
CE1 / # exit
```

### 1\. システム基本設定

#### ホスト名設定

```bash
/configure system name "PE1"
```

**確認コマンド:**

  - `show chassis`
  - `show chassis detail`

#### ログインアイドルタイムアウト設定

```bash
/configure system login-control idle-timeout none
```

**確認コマンド:**

  - `show system management-interface configuration-sessions`

#### NTP設定 (\*172.16.[pod\#].10)

```bash
/configure system time zone non-standard name "jst"
/configure system time zone non-standard offset "09:00"
/configure system time ntp admin-state enable
/configure system time ntp peer 172.16.1.10 router-instance "management" version 4
/configure system time ntp peer 172.16.1.10 router-instance "management" prefer true
```

**確認コマンド:**

  - `show time`
  - `show system ntp`
  - `show system ntp details`
  - `show system ntp peers`

-----

### 2\. ロギング・通知設定

#### ロギングフィルタ設定

```bash
/configure log filter "1001" named-entry "10" description "Collect only events of major severity or higher"
/configure log filter "1001" named-entry "10" action forward
/configure log filter "1001" named-entry "10" match severity gte major
```

**確認コマンド:**

  - `show log log-id`
  - `show log event-control`
  - `show log log-id "99"`
  - `show log log-id "100"`

#### SNMP TRAP設定 (\*172.16.[pod\#].10)

```bash
/configure log snmp-trap-group "10" trap-target "snmptrapd" address 172.16.1.10
/configure log snmp-trap-group "10" trap-target "snmptrapd" port 162
/configure log snmp-trap-group "10" trap-target "snmptrapd" version snmpv2c
/configure log snmp-trap-group "10" trap-target "snmptrapd" notify-community "public"
/configure log log-id "10" source main true
/configure log log-id "10" source security true
/configure log log-id "10" source change true
/configure log log-id "10" { destination snmp }
```

**確認コマンド:**

  - `show log log-id`
  - `tools perform log test-event`

#### SYSLOG設定 (\*172.16.[pod\#].10)

```bash
/configure log syslog "1" address 172.16.1.10
/configure log syslog "1" port 514
/configure log log-id "20" source main true
/configure log log-id "20" source security true
/configure log log-id "20" source change true
/configure log log-id "20" destination syslog "1"
```

**確認コマンド:**

  - `show log log-id`
  - `tools perform log test-event`

#### ターミナルへのログ出力

```bash
/configure log log-id "22" admin-state enable
/configure log log-id "22" source main true
/configure log log-id "22" { destination cli }
```

**確認コマンド:**

  - `show log log-id`
  - `tools perform log subscribe-to log-id "22"`
  - `tools perform log unsubscribe-from log-id "22"`

-----

### 3\. 物理ポート・カード設定

#### インターフェースカード設定

```bash
/configure card 1 card-type imm36-qsfpdd
/configure card 1 mda 1 mda-type m36-qsfpdd
```

**確認コマンド:**

  - `show card`
  - `show card detail`
  - `show card A`
  - `show card A detail`
  - `show card 1`
  - `show card 1 detail`
  - `show mda`
  - `show mda detail`

#### ポート設定(CE側)

```bash
/configure port 1/1/c1 admin-state enable
/configure port 1/1/c1 description "To CE1"
/configure port 1/1/c1 connector breakout c1-100g

/configure port 1/1/c1/1 admin-state enable
/configure port 1/1/c1/1 description "To CE1"
/configure port 1/1/c1/1 ethernet mode access
/configure port 1/1/c1/1 ethernet encap-type dot1q
/configure port 1/1/c1/1 ethernet mtu 9200
```

**確認コマンド:**

  - `show port`
  - `show port detail`

#### ポート設定(CORE側)

```bash
/configure port 1/1/c2 admin-state enable
/configure port 1/1/c2 description "To P1"
/configure port 1/1/c2 connector breakout c1-400g
/configure port 1/1/c2/1 admin-state enable
/configure port 1/1/c2/1 description "To P1"
/configure port 1/1/c2/1 ethernet mode network
/configure port 1/1/c2/1 ethernet encap-type null
/configure port 1/1/c2/1 ethernet mtu 9200

/configure port 1/1/c3 admin-state enable
/configure port 1/1/c3 description "To PE2"
/configure port 1/1/c3 connector breakout c1-400g

/configure port 1/1/c3/1 admin-state enable
/configure port 1/1/c3/1 description "To PE2"
/configure port 1/1/c3/1 ethernet mode network
/configure port 1/1/c3/1 ethernet encap-type null
/configure port 1/1/c3/1 ethernet mtu 9200
```

**確認コマンド:**

  - `show port`
  - `show port detail`

-----

### 4\. LLDP 設定

```bash
/configure port 1/1/c2/1 ethernet lldp dest-mac nearest-bridge receive true
/configure port 1/1/c2/1 ethernet lldp dest-mac nearest-bridge transmit true
/configure port 1/1/c2/1 ethernet lldp dest-mac nearest-bridge tx-tlvs port-desc true
/configure port 1/1/c2/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-cap true
/configure port 1/1/c2/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address system admin-state enable

/configure port 1/1/c3/1 ethernet lldp dest-mac nearest-bridge receive true
/configure port 1/1/c3/1 ethernet lldp dest-mac nearest-bridge transmit true
/configure port 1/1/c3/1 ethernet lldp dest-mac nearest-bridge tx-tlvs port-desc true
/configure port 1/1/c3/1 ethernet lldp dest-mac nearest-bridge tx-tlvs sys-cap true
/configure port 1/1/c3/1 ethernet lldp dest-mac nearest-bridge tx-mgmt-address system admin-state enable
```

**確認コマンド:**

  - `show system lldp`
  - `show system lldp neighbor`

-----

### 5\. L3 ネットワーク設定 (Base Router)

#### systemループバック & ASN 設定

```bash
/configure router "Base" autonomous-system 65001
/configure router "Base" interface "system" ipv4 primary address 10.0.0.1
/configure router "Base" interface "system" ipv4 primary prefix-length 32
/configure router "Base" interface "system" ipv6 address 1000::1 prefix-length 128
```

**確認コマンド:**

  - `show router interface system`
  - `show router interface system detail`

#### L3インターフェース設定

```bash
/configure router "Base" interface "to_P1" port 1/1/c2/1
/configure router "Base" interface "to_P1" ipv6 address 2000:2:1::1 prefix-length 64
/configure router "Base" interface "to_PE2" port 1/1/c3/1
/configure router "Base" interface "to_PE2" ipv6 address 2000:3:1::1 prefix-length 64
```

**確認コマンド:**

  - `show router interface`
  - `show router interface detail`
  - `ping router Base 2000:2:1::2`
  - `ping router Base 2000:3:1::2`

-----

### 6\. BFD 設定

```bash
/configure router "Base" interface "to_P1" ipv6 bfd admin-state enable
/configure router "Base" interface "to_P1" ipv6 bfd transmit-interval 100
/configure router "Base" interface "to_P1" ipv6 bfd receive 100
/configure router "Base" interface "to_P1" ipv6 bfd multiplier 3

/configure router "Base" interface "to_PE2" ipv6 bfd admin-state enable
/configure router "Base" interface "to_PE2" ipv6 bfd transmit-interval 100
/configure router "Base" interface "to_PE2" ipv6 bfd receive 100
/configure router "Base" interface "to_PE2" ipv6 bfd multiplier 3
```

**確認コマンド:**

  - `show router bfd session`
  - `show router bfd session detail`

-----

### 7\. 経路ポリシー設定

```bash
/configure { policy-options prefix-list "system_ipv4" prefix 10.0.0.1/32 type exact }
/configure { policy-options prefix-list "system_ipv4" prefix 10.0.0.2/32 type exact }
/configure { policy-options prefix-list "system_ipv4" prefix 10.0.0.3/32 type exact }

/configure { policy-options prefix-list "system_ipv6" prefix 1000::/64 type longer }
/configure policy-options policy-statement "system_ip" entry 10 from prefix-list ["system_ipv6"]
/configure policy-options policy-statement "system_ip" entry 10 action action-type accept
/configure policy-options policy-statement "system_ip" entry 20 from prefix-list ["system_ipv4"]
/configure policy-options policy-statement "system_ip" entry 20 action action-type accept
```

**確認コマンド:**

  - `show router policy`
  - `show router policy "system_ip"`

-----

### 8\. BGP 設定 (Global/Underlay/Overlay)

#### BGP(グローバル)設定

```bash
/configure router "Base" bgp admin-state enable
/configure router "Base" bgp loop-detect off
/configure router "Base" bgp min-route-advertisement 1
/configure router "Base" bgp bfd-liveness true
/configure router "Base" bgp ebgp-default-reject-policy import false
/configure router "Base" bgp ebgp-default-reject-policy export false
/configure router "Base" bgp rapid-update evpn true
```

**確認コマンド:**

  - `show router bgp summary`

#### BGP(アンダーレイ)設定

```bash
/configure router "Base" bgp group "underlay" family ipv4 true
/configure router "Base" bgp group "underlay" family ipv6 true
/configure router "Base" bgp group "underlay" export policy ["system_ip"]
/configure router "Base" bgp group "underlay" extended-nh-encoding ipv4 true
/configure router "Base" bgp group "underlay" advertise-ipv6-next-hops ipv4 true
/configure router "Base" bgp neighbor "2000:2:1::2" group "underlay"
/configure router "Base" bgp neighbor "2000:2:1::2" peer-as 65011
/configure router "Base" bgp neighbor "2000:3:1::2" group "underlay"
/configure router "Base" bgp neighbor "2000:3:1::2" peer-as 65002
```

**確認コマンド:**

  - `show router bgp summary`
  - `show router bgp neighbor`
  - `show router bgp neighbor "1000::2" advertised-routes`
  - `show router bgp neighbor "1000::2" received-routes`
  - `show router bgp routes`
  - `show router route-table`
  - `show router route-table ipv6`

#### BGP(オーバーレイ)設定

```bash
/configure router "Base" bgp group "overlay" multihop 255
/configure router "Base" bgp group "overlay" family evpn true
/configure router "Base" bgp group "overlay" local-as as-number 65001
/configure router "Base" bgp neighbor "1000::2" group "overlay"
/configure router "Base" bgp neighbor "1000::2" peer-as 65002
/configure router "Base" bgp neighbor "1000::3" admin-state enable
/configure router "Base" bgp neighbor "1000::3" group "overlay"
/configure router "Base" bgp neighbor "1000::3" peer-as 65003
```

**確認コマンド:**

  - `show router bgp summary`
  - `show router bgp neighbor`
  - `show router bgp neighbor "1000::2" advertised-routes`
  - `show router bgp neighbor "1000::2" received-routes`
  - `show router bgp routes`
  - `show router bgp routes evpn mac`
  - `show router route-table`
  - `show router route-table ipv6`

-----

### 9\. L2VPN サービス設定 (VPLS)

```bash
/configure service vpls "L2VPN" admin-state enable
/configure service vpls "L2VPN" service-id 10
/configure service vpls "L2VPN" customer "1"
/configure service vpls "L2VPN" vxlan instance 1 vni 10
/configure { service vpls "L2VPN" routed-vpls }
/configure service vpls "L2VPN" bgp 1 route-distinguisher "65002:10"
/configure service vpls "L2VPN" bgp 1 route-target export "target:65000:10"
/configure service vpls "L2VPN" bgp 1 route-target import "target:65000:10"
/configure service vpls "L2VPN" bgp-evpn evi 10
/configure service vpls "L2VPN" bgp-evpn routes mac-ip advertise true
/configure service vpls "L2VPN" bgp-evpn vxlan 1 admin-state enable
/configure service vpls "L2VPN" bgp-evpn vxlan 1 vxlan-instance 1
/configure service vpls "L2VPN" sap 1/1/c1/1:10 admin-state enable
```

**確認コマンド:**

  - `show service service-using`
  - `show service sap-using`
  - `show service id "L2VPN" all`
  - `show service id "L2VPN" fdb detail`

-----

### 10\. 最終疎通確認

CE（Docker コンテナ）から、他の拠点への疎通確認を実施します。

```bash
docker exec clab-pod01-CE1 ping 192.168.10.2
docker exec clab-pod01-CE1 ping 192.168.10.10
docker exec clab-pod01-CE1 ping 192.168.10.3
```
