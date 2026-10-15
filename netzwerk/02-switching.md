# Netzwerk Phase 2 – Switching

**Dauer:** 4–6 Wochen · **Voraussetzung:** Phase 1

---

## 1. Funktionsweise eines Switches

- [ ] MAC-Adresstabelle (CAM): Lernen (Quell-MAC), Weiterleiten (Ziel-MAC), **Flooding** bei unbekannter Ziel-MAC/Broadcast
- [ ] Aging Time (Standard 300 s)
- [ ] Store-and-Forward vs. Cut-Through (im RZ für Low Latency)
- [ ] Layer-2- vs. Layer-3-Switch
- [ ] Switching-Hardware: ASIC, TCAM, Puffer (Buffer) – warum „Deep Buffer“ Switches für Storage wichtig sind

## 2. VLANs

- [ ] Zweck: Broadcastdomänen trennen, Sicherheit, Organisation
- [ ] VLAN-Bereich 1–4094 (1 = Default, 1002–1005 reserviert bei Cisco), Extended 1006–4094
- [ ] **Access-Port** (ein VLAN, untagged) vs. **Trunk-Port** (mehrere VLANs, getaggt mit **802.1Q**)
- [ ] 802.1Q-Tag: TPID 0x8100, PCP (3 Bit, QoS), DEI, **VLAN-ID (12 Bit)**
- [ ] **Native VLAN** (untagged auf Trunk) – Sicherheitsrisiko, auf ungenutztes VLAN legen
- [ ] **Allowed VLANs** auf Trunks immer explizit beschränken
- [ ] DTP abschalten (`switchport nonegotiate`)
- [ ] VTP kennen (und warum man es meist in `transparent`/`off` betreibt)
- [ ] Private VLANs (isolated/community) – Grundzüge, für Hosting-Umgebungen
- [ ] QinQ (802.1ad) – Grundzüge, Provider

## 3. Spanning Tree

- [ ] Warum: Layer-2-Loops → Broadcast-Sturm, MAC-Flapping, Netz steht
- [ ] **STP 802.1D**: Root-Bridge-Wahl (niedrigste Bridge-ID = Priorität [Standard 32768] + MAC)
- [ ] Portrollen: Root Port, Designated Port, Non-Designated/Blocked; Pfadkosten (1G = 4, 10G = 2 kurzer Modus)
- [ ] Portzustände: Blocking → Listening → Learning → Forwarding (bis 50 s!)
- [ ] **RSTP 802.1w**: schnelle Konvergenz, Rollen Alternate/Backup, Port-Typen edge/point-to-point
- [ ] **MSTP 802.1s**: mehrere Instanzen, Regionen
- [ ] Cisco PVST+ / Rapid-PVST+
- [ ] **Root Bridge bewusst festlegen** (`spanning-tree vlan X root primary` / Priorität 4096)
- [ ] Schutzmechanismen: **PortFast** (Edge-Ports), **BPDU Guard**, Root Guard, Loop Guard, BPDU Filter (Vorsicht!)
- [ ] Topology Change Notifications (TCN) und warum häufige TCNs MAC-Tabellen leeren

## 4. Link-Aggregation

- [ ] **LACP 802.3ad / 802.1AX** vs. statisches Bündeln (`mode on`) vs. PAgP (Cisco)
- [ ] LACP-Modi: active/passive; mindestens eine Seite active
- [ ] Load-Balancing-Hash (src/dst MAC, IP, Port) – **eine einzelne Verbindung nutzt nur einen Link!**
- [ ] Gleiche Speed/Duplex/VLAN-Konfiguration auf allen Mitgliedsports
- [ ] **MLAG / vPC / MC-LAG**: Server hängt mit LACP an **zwei** Switches – Redundanz ohne STP-Blockierung
  - Peer-Link, Peer-Keepalive, Orphan Ports, Split-Brain
- [ ] Gegenstück am Server: Linux-Bonding Mode 4 (802.3ad), Windows NIC Teaming (LACP), VMware LAG

## 5. Rechenzentrums-Topologien

- [ ] **Klassisch 3-Tier**: Core – Aggregation/Distribution – Access; Nord-Süd-Verkehr; STP-lastig
- [ ] **Spine-Leaf (Clos)**: jeder Leaf mit jedem Spine verbunden, keine Leaf-Leaf-/Spine-Spine-Links
  - Gleiche Hop-Anzahl zwischen allen Servern, Ost-West-optimiert
  - Underlay geroutet (L3, ECMP, BGP/OSPF), L2 über **VXLAN/EVPN** (→ Phase 3)
  - Skalierung: mehr Spines = mehr Bandbreite, mehr Leafs = mehr Ports
  - **Oversubscription** berechnen: z. B. 48 × 25G Downlink = 1200G, 6 × 100G Uplink = 600G → 2:1
- [ ] **Top-of-Rack (ToR)** vs. End-of-Row (EoR) / Middle-of-Row
- [ ] Border-Leafs (Anbindung an Firewalls/WAN/Internet), Service-Leafs (Load Balancer)
- [ ] OOB-Management-Netz (separate Switches für iDRAC/iLO/Konsolen-Server)

## 6. Switch-Sicherheit

- [ ] **Port Security** (max. MAC-Anzahl, sticky, Violation: shutdown/restrict/protect)
- [ ] **DHCP Snooping** (trusted/untrusted Ports, Binding-Tabelle)
- [ ] **Dynamic ARP Inspection** (nutzt DHCP-Snooping-Tabelle)
- [ ] IP Source Guard
- [ ] **Storm Control** (Broadcast/Multicast-Limits)
- [ ] **802.1X** Port-Authentifizierung (Supplicant, Authenticator, Authentication Server/RADIUS), MAB
- [ ] Ungenutzte Ports: `shutdown` + in Parking-VLAN
- [ ] errdisable verstehen und beheben (`show interfaces status err-disabled`, `errdisable recovery`)

## 7. Nachbarerkennung

- [ ] **LLDP** (herstellerneutral, IEEE 802.1AB) und **CDP** (Cisco)
- [ ] Nutzen: Verkabelung prüfen („Hängt der Server wirklich an Port 12?“)
- [ ] Am Server: `lldpctl` (Paket `lldpd`), VMware zeigt CDP/LLDP im vSwitch

## 8. Befehle

### Cisco IOS – Anzeigen
```
show running-config interface Gi1/0/12
show interfaces status
show interfaces Gi1/0/12
show interfaces Gi1/0/12 switchport
show interfaces trunk
show interfaces counters errors
show interfaces transceiver detail        ! Rx/Tx-Power der Optiken
show vlan brief
show mac address-table
show mac address-table address 0050.5689.1a2b
show mac address-table interface Gi1/0/12
show spanning-tree
show spanning-tree vlan 10
show spanning-tree root
show spanning-tree detail | include ieee|occur|from|is exec   ! Wo kommen TCNs her?
show etherchannel summary
show lacp neighbor
show lldp neighbors [detail]
show cdp neighbors detail
show port-security interface Gi1/0/5
show ip dhcp snooping binding
show errdisable recovery
show logging | include Gi1/0/12
```

### Cisco IOS – Konfiguration
```
vlan 110
 name SRV-PROD
!
interface GigabitEthernet1/0/12
 description srv-web01 eth0 | Rack B07 HE21
 switchport mode access
 switchport access vlan 110
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
 storm-control broadcast level 1.00
 no shutdown
!
interface range Gi1/0/47 - 48
 channel-group 1 mode active
!
interface Port-channel1
 description Uplink zu dist-sw01
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 110,120,130
!
spanning-tree mode rapid-pvst
spanning-tree vlan 110,120,130 priority 4096
```

### NX-OS (vPC-Beispiel)
```
feature lacp
feature vpc
vpc domain 10
  peer-keepalive destination 10.255.0.2 source 10.255.0.1 vrf management
interface port-channel10
  vpc peer-link
interface port-channel20
  description srv-esx01
  switchport mode trunk
  vpc 20
show vpc brief
show vpc consistency-parameters interface port-channel 20
```

### Linux-Server-Seite
```bash
cat /proc/net/bonding/bond0      # Bonding-Status, LACP-Partner
ip -d link show bond0
bridge vlan show
lldpctl                          # an welchem Switchport hänge ich?
```

## 9. Typische Störungen & Diagnose

| Symptom | Mögliche Ursache | Prüfen mit |
|---|---|---|
| Port up, kein Traffic | falsches VLAN, Port im errdisable, STP blocking | `show int status`, `show spanning-tree int` |
| Netz „steht“, CPU hoch | Layer-2-Loop | `show spanning-tree detail`, MAC-Flapping-Logs |
| Langsam, CRC-Fehler | Kabel/Optik/Duplex | `show interfaces`, `show int transceiver` |
| Port-Channel teilweise down | Konfig-Mismatch, LACP nicht aktiv auf Gegenseite | `show etherchannel summary` (Flags!) |
| Server erreicht Gateway sporadisch | doppelte IP, MLAG-Inkonsistenz | `show mac address-table`, `arping` |
| Port geht auf errdisable | BPDU Guard (jemand hat Switch angeschlossen), Port Security | `show logging`, `show int status err-disabled` |

## 10. Praxisaufgaben

- [ ] Packet Tracer / GNS3: 3 Switches im Dreieck, Rapid-PVST, Root Bridge festlegen, Link abziehen und Konvergenz beobachten
- [ ] LACP-Bündel zwischen zwei Switches + Bonding auf einer Linux-VM
- [ ] Bewusst einen Loop erzeugen (ohne STP) und Auswirkungen in Wireshark sehen – **nur im Labor!**
- [ ] Im Betrieb: Ein Server-Port-Patch-Auftrag komplett begleiten (Ticket → Patchen → Konfig → Test → Doku)
- [ ] Oversubscription eines Leaf-Switches aus eurem RZ berechnen

## Meine Notizen

