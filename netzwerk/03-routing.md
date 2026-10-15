# Netzwerk Phase 3 – Routing

**Dauer:** 6–8 Wochen · **Voraussetzung:** Phase 2

---

## 1. Routing-Grundlagen

- [ ] Routing-Tabelle lesen: Präfix, Next Hop, Interface, AD, Metrik, Quelle (C, L, S, O, B …)
- [ ] **Longest Prefix Match** – die spezifischste Route gewinnt, *dann* AD, *dann* Metrik
- [ ] **Administrative Distanz** (Cisco):

| Quelle | AD |
|---|---|
| Connected | 0 |
| Static | 1 |
| eBGP | 20 |
| EIGRP (intern) | 90 |
| OSPF | 110 |
| IS-IS | 115 |
| RIP | 120 |
| iBGP | 200 |

- [ ] Default-Route, Floating Static Route (höhere AD als Backup)
- [ ] Rekursives Routing (Next Hop wird über andere Route aufgelöst)
- [ ] **ECMP** – Equal Cost Multi-Path, Grundlage von Spine-Leaf
- [ ] Control Plane vs. Data Plane vs. Management Plane; RIB vs. FIB; CEF

## 2. Inter-VLAN-Routing

- [ ] Router-on-a-Stick (Subinterfaces mit `encapsulation dot1Q`)
- [ ] **SVI** auf L3-Switch (`interface Vlan110`) – Standard im RZ
- [ ] Anycast-Gateway (im EVPN: gleiches Gateway auf jedem Leaf)

## 3. First Hop Redundancy

- [ ] **HSRP** (Cisco): Virtual IP, Priorität, Preempt, Active/Standby, Tracking
- [ ] **VRRP** (Standard, RFC 5798): Master/Backup
- [ ] GLBP (Cisco, mit Load Balancing)
- [ ] Bei MLAG/vPC: beide Gateways aktiv (Peer-Gateway / VARP bei Arista)

## 4. OSPF

- [ ] Link-State, Dijkstra/SPF, LSDB
- [ ] Router-ID (höchste Loopback-IP oder manuell – **immer manuell setzen**)
- [ ] Areas: **Area 0 = Backbone**, ABR, ASBR; Stub/Totally Stubby/NSSA
- [ ] Nachbarschaft: Hello/Dead Timer (10/40), gleiche Area, Subnetz, MTU, Auth, Timer müssen passen
- [ ] Zustände: Down → Init → 2-Way → ExStart → Exchange → Loading → **Full**
- [ ] DR/BDR auf Broadcast-Netzen; `ip ospf network point-to-point` auf Transfernetzen
- [ ] Kosten = Referenzbandbreite / Interfacebandbreite → **Referenzbandbreite im RZ erhöhen** (`auto-cost reference-bandwidth 100000`)
- [ ] LSA-Typen 1, 2, 3, 5, 7 (Grundzüge)
- [ ] Passive Interfaces, Authentifizierung
- [ ] Typischer Fehler „stuck in ExStart“ = MTU-Mismatch

## 5. BGP – im RZ besonders wichtig

- [ ] Path-Vector-Protokoll, TCP 179, **Autonomous System** (AS), öffentliche/private ASNs (64512–65534, 4-Byte: 4200000000+)
- [ ] **eBGP** (zwischen AS, TTL 1) vs. **iBGP** (innerhalb AS, Full-Mesh oder Route-Reflector, next-hop-self)
- [ ] Session-Zustände: Idle → Connect → Active → OpenSent → OpenConfirm → **Established**
  - „Active“ heißt **nicht** gut – es heißt, die Session kommt nicht hoch!
- [ ] Routen announcen: `network`-Statement (muss exakt in RIB stehen) oder Redistribution
- [ ] **Best-Path-Selection** (Reihenfolge merken – „We Love Oranges AS Oranges Mean Pure Refreshment“):
  1. Weight (Cisco, lokal, höher besser)
  2. Local Preference (im AS, höher besser)
  3. Originated locally
  4. AS-Path (kürzer besser)
  5. Origin (IGP < EGP < incomplete)
  6. MED (niedriger besser)
  7. eBGP vor iBGP
  8. niedrigste IGP-Metrik zum Next Hop
  9. … Router-ID
- [ ] Policies: Prefix-Lists, Route-Maps, AS-Path-Filter, Communities (z. B. Blackhole-Community für DDoS)
- [ ] Prefix-Limits (max-prefix), BFD für schnelle Ausfallerkennung
- [ ] Internet-Anbindung: Upstream, Peering (IXP wie DE-CIX), Transit, RPKI/ROA (Route-Origin-Validation), IRR
- [ ] **BGP im RZ-Underlay** (RFC 7938): jeder Leaf eigenes AS oder alle Leafs gleiche AS, eBGP zu den Spines, ECMP

## 6. Weitere Themen

- [ ] **IS-IS** (Grundzüge, verbreitet in RZ- und Provider-Netzen)
- [ ] EIGRP (für CCNA-Wissen, im RZ selten)
- [ ] **VRF / VRF-lite**: getrennte Routing-Tabellen, Mandantentrennung, Management-VRF
- [ ] Route-Leaking zwischen VRFs
- [ ] **NAT/PAT**: Static NAT, Dynamic NAT, PAT/Overload, inside/outside local/global
- [ ] Policy Based Routing (PBR)
- [ ] MPLS-Grundzüge (Labels, LDP, L3VPN) – für das Verständnis von Provider-Anbindungen
- [ ] Multicast-Grundzüge (IGMP, PIM) – falls im RZ benötigt

## 7. VXLAN / EVPN – das moderne RZ-Fabric

- [ ] Problem: L2-Domänen über ein geroutetes Underlay spannen, mehr als 4094 Segmente
- [ ] **VXLAN**: MAC-in-UDP, Port **4789**, **VNI** (24 Bit = 16 Mio. Segmente), **VTEP** (Tunnel-Endpunkt, meist der Leaf)
- [ ] Overhead 50 Byte → **Underlay-MTU auf mind. 9000 / 9216 setzen**
- [ ] Flood-and-Learn (alt) vs. **BGP EVPN** als Control-Plane (heute Standard)
- [ ] EVPN-Routentypen: **Type 2** (MAC/IP), **Type 3** (Inclusive Multicast / BUM), **Type 5** (IP-Präfix)
- [ ] L2VNI (Bridging) vs. L3VNI (Routing zwischen VRFs, symmetrisches IRB)
- [ ] Distributed Anycast Gateway
- [ ] ARP-Suppression

## 8. Befehle

### Cisco – Anzeigen
```
show ip route
show ip route 10.20.30.40          ! welche Route greift?
show ip route ospf | bgp | static
show ip cef 10.20.30.40
show ip protocols
show ip ospf neighbor
show ip ospf interface brief
show ip ospf database
show ip bgp summary                 ! Neighbor, State/PfxRcd, Up/Down
show ip bgp                         ! BGP-Tabelle (*> = best/valid)
show ip bgp 203.0.113.0/24
show ip bgp neighbors 192.0.2.1 advertised-routes
show ip bgp neighbors 192.0.2.1 received-routes   ! (soft-reconfig inbound nötig)
show standby brief
show vrrp brief
show vrf
show ip route vrf KUNDE-A
show ip nat translations
show ip nat statistics
show bfd neighbors
! NX-OS / EVPN
show nve peers
show nve vni
show bgp l2vpn evpn summary
show bgp l2vpn evpn
show l2route evpn mac all
```

### Cisco – Konfiguration (Beispiele)
```
ip route 0.0.0.0 0.0.0.0 192.0.2.1
ip route 0.0.0.0 0.0.0.0 198.51.100.1 250          ! Floating Static
!
router ospf 1
 router-id 10.255.255.1
 auto-cost reference-bandwidth 100000
 passive-interface default
 no passive-interface Ethernet1/1
 network 10.0.0.0 0.0.0.255 area 0
!
router bgp 65001
 bgp router-id 10.255.255.1
 neighbor 192.0.2.1 remote-as 64500
 neighbor 192.0.2.1 description Upstream-ISP-A
 neighbor 192.0.2.1 prefix-list OUT-ALLOWED out
 neighbor 192.0.2.1 maximum-prefix 1000000
 network 203.0.113.0 mask 255.255.255.0
!
ip prefix-list OUT-ALLOWED seq 10 permit 203.0.113.0/24
!
interface Vlan110
 ip address 10.110.0.2 255.255.255.0
 standby 110 ip 10.110.0.1
 standby 110 priority 110
 standby 110 preempt
```

### Linux als Router / Diagnose
```bash
sysctl net.ipv4.ip_forward              # Forwarding an?
ip route add 10.50.0.0/16 via 10.0.0.1
ip rule ; ip route show table all       # Policy Routing
ip vrf ; ip route show vrf mgmt
vtysh -c "show ip bgp summary"          # FRRouting
```

## 9. Praxisaufgaben

- [ ] OSPF mit 2 Areas (Packet Tracer/GNS3), Summarization am ABR
- [ ] Zwei AS per eBGP verbinden, Local Preference für bevorzugten Upstream setzen, Ausfall simulieren
- [ ] **Containerlab** + Arista cEOS oder FRRouting: 2 Spines, 4 Leafs, eBGP-Underlay, VXLAN-EVPN mit einem L2VNI und einem L3VNI (Fortgeschritten – Projekt in [`labor/`](../labor/README.md))
- [ ] Im Betrieb: `show ip bgp summary` auf einem Border-Router mit dem Ausbilder durchgehen – wie viele Präfixe von welchem Upstream?
- [ ] Erklären können: Was passiert, wenn ein Spine ausfällt?

## Meine Notizen

