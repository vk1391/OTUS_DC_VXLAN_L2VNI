# VxLAN. EVPN L2
## Задание:
- Настроите BGP peering между Leaf и Spine в AF l2vpn evpn
- Настроите связанность между клиентами в первой зоне и убедитесь в её наличии

underlay - iBGP
![alt-dtp](https://github.com/vk1391/OTUS_DC_VXLAN_L2VNI/blob/main/vxlan_l2vni.jpg)

### Настроите BGP peering между Leaf и Spine в AF l2vpn evpn
 - конфигурация vxlan evpn Leaf1:
```
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100
   vxlan flood vtep 101.13.13.13
!
ip routing
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 11.11.11.11
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.11.1 remote-as 101
   neighbor 10.1.11.1 next-hop-self
   neighbor 10.1.11.1 bfd
   neighbor 10.2.11.1 remote-as 101
   neighbor 10.2.11.1 next-hop-self
   neighbor 10.2.11.1 bfd
   neighbor 101.13.13.13 remote-as 101
   neighbor 101.13.13.13 update-source Loopback1
   neighbor 101.13.13.13 send-community extended
   redistribute connected route-map redist
   !
   vlan 10
      rd 101.13.13.13:1
      route-target both 101:1
      redistribute learned
   !
   address-family evpn
      neighbor 101.13.13.13 activate
   !
   address-family ipv4
      neighbor 10.1.11.1 activate
      neighbor 10.2.11.1 activate
      no neighbor 101.13.13.13 activate
```
- конфигурация vxlan evpn Leaf3:
```
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100
   vxlan flood vtep 101.11.11.11
!
ip routing
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 13.13.13.13
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.13.1 remote-as 101
   neighbor 10.1.13.1 bfd
   neighbor 10.2.13.1 remote-as 101
   neighbor 10.2.13.1 bfd
   neighbor 101.11.11.11 remote-as 101
   neighbor 101.11.11.11 update-source Loopback1
   neighbor 101.11.11.11 send-community extended
   redistribute connected route-map redist
   !
   vlan 10
      rd 101.11.11.11:1
      route-target both 101:1
      redistribute learned
   !
   address-family evpn
      neighbor 101.11.11.11 activate
   !

   address-family ipv4
      no neighbor 101.11.11.11 activate
```
## Настроите связанность между клиентами в первой зоне и убедитесь в её наличии
- таблица соседства bgp evpn leaf1:
```
localhost(config-router-bgp)#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 11.11.11.11, local AS number 101
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  101.13.13.13 4 101            26943     26945    0    0 19:06:40 Estab   1      1
```
- информация evpn leaf1:
```
localhost#sh bgp evpn route-type imet detail
BGP routing table information for VRF default
Router identifier 11.11.11.11, local AS number 101
BGP routing table entry for imet 101.11.11.11, Route Distinguisher: 101.13.13.13:1
 Paths: 1 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100
      PMSI Tunnel: Ingress Replication, MPLS Label: 100, Leaf Information Required: false, Tunnel ID: 101.11.11.11
BGP routing table entry for imet 101.13.13.13, Route Distinguisher: 101.11.11.11:1
 Paths: 1 available
  Local
    101.13.13.13 from 101.13.13.13 (13.13.13.13)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, internal, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100
      PMSI Tunnel: Ingress Replication, MPLS Label: 100, Leaf Information Required: false, Tunnel ID: 101.13.13.13

localhost#sh bgp evpn route-type mac-ip detail 
BGP routing table information for VRF default
Router identifier 11.11.11.11, local AS number 101
BGP routing table entry for mac-ip 0050.7966.6806, Route Distinguisher: 101.13.13.13:1
 Paths: 1 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6808, Route Distinguisher: 101.11.11.11:1
 Paths: 1 available
  Local
    101.13.13.13 from 101.13.13.13 (13.13.13.13)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, internal, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 2aba.cc65.b25d, Route Distinguisher: 101.11.11.11:1
 Paths: 1 available
  Local
    101.13.13.13 from 101.13.13.13 (13.13.13.13)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, internal, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 8a60.aca3.a472, Route Distinguisher: 101.13.13.13:1
 Paths: 1 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100 ESI: 0000:0000:0000:0000:0000
```
- таблица соседства bgp evpn leaf3:
```
localhost#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 13.13.13.13, local AS number 101
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  101.11.11.11 4 101            27219     27215    0    0 19:18:41 Estab   1      1
```
- информация evpn leaf3:
```
localhost#sh bgp evpn route-type imet detail 
BGP routing table information for VRF default
Router identifier 13.13.13.13, local AS number 101
BGP routing table entry for imet 101.11.11.11, Route Distinguisher: 101.13.13.13:1
 Paths: 1 available
  Local
    101.11.11.11 from 101.11.11.11 (11.11.11.11)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, internal, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100
      PMSI Tunnel: Ingress Replication, MPLS Label: 100, Leaf Information Required: false, Tunnel ID: 101.11.11.11
BGP routing table entry for imet 101.13.13.13, Route Distinguisher: 101.11.11.11:1
 Paths: 1 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100
      PMSI Tunnel: Ingress Replication, MPLS Label: 100, Leaf Information Required: false, Tunnel ID: 101.13.13.13

localhost#sh bgp evpn route-type mac-ip detail
BGP routing table information for VRF default
Router identifier 13.13.13.13, local AS number 101
BGP routing table entry for mac-ip 0050.7966.6806, Route Distinguisher: 101.13.13.13:1
 Paths: 1 available
  Local
    101.11.11.11 from 101.11.11.11 (11.11.11.11)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, internal, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6808, Route Distinguisher: 101.11.11.11:1
 Paths: 1 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best
      Extended Community: Route-Target-AS:101:1 TunnelEncap:tunnelTypeVxlan
      VNI: 100 ESI: 0000:0000:0000:0000:0000
```
- ping vpc - vpc8:
```
VPCS> ping 10.0.0.2

84 bytes from 10.0.0.2 icmp_seq=1 ttl=64 time=32.811 ms
84 bytes from 10.0.0.2 icmp_seq=2 ttl=64 time=27.426 ms
84 bytes from 10.0.0.2 icmp_seq=3 ttl=64 time=33.483 ms
84 bytes from 10.0.0.2 icmp_seq=4 ttl=64 time=27.680 ms
84 bytes from 10.0.0.2 icmp_seq=5 ttl=64 time=45.079 ms
```
