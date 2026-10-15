# Netzwerk Phase 1 – Grundlagen & Adressierung

**Dauer:** 4–6 Wochen · **Voraussetzung:** [00-fundament](../00-fundament.md)

---

## 1. Ethernet (Layer 2)

- [ ] **MAC-Adresse**: 48 Bit, 6 Byte hex, erste 3 Byte = OUI (Hersteller), Formate `00:1A:2B:3C:4D:5E` / `001a.2b3c.4d5e` (Cisco)
- [ ] Unicast / Multicast (LSB im ersten Byte = 1) / Broadcast `ff:ff:ff:ff:ff:ff`
- [ ] **Frame-Aufbau**: Präambel, Ziel-MAC, Quell-MAC, (802.1Q-Tag 4 Byte), EtherType (0x0800 IPv4, 0x86DD IPv6, 0x0806 ARP, 0x8100 VLAN), Payload, FCS (CRC)
- [ ] **MTU**: Standard 1500 Byte Payload; **Jumbo Frames 9000** (im RZ für Storage/vMotion üblich) – MTU muss auf dem *gesamten* Pfad passen!
- [ ] Kollisionsdomäne (pro Switchport) vs. Broadcastdomäne (pro VLAN)
- [ ] Duplex (Half/Full) & Speed, Autonegotiation, **Duplex-Mismatch** als klassische Fehlerursache (Symptom: Late Collisions, CRC-Fehler, langsam)
- [ ] Interface-Fehlerzähler deuten: CRC (meist L1: Kabel/Optik), Input Errors, Runts, Giants, Drops/Discards (Überlast/Puffer)

## 2. IPv4

### 2.1 Grundlagen
- [ ] 32 Bit, 4 Oktette, Netz- und Hostanteil, Subnetzmaske, CIDR-Präfix
- [ ] Spezielle Bereiche:

| Bereich | Zweck |
|---|---|
| 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | Privat (RFC 1918) |
| 127.0.0.0/8 | Loopback |
| 169.254.0.0/16 | APIPA / Link-Local (kein DHCP erhalten!) |
| 100.64.0.0/10 | Carrier-Grade NAT |
| 224.0.0.0/4 | Multicast |
| 0.0.0.0/0 | Default-Route |
| 255.255.255.255 | Limited Broadcast |
| 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24 | Dokumentation (für Beispiele!) |

### 2.2 Subnetting – **muss im Kopf sitzen**

| Präfix | Maske | Block | Hosts | Typischer Einsatz |
|---|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 | klassisches Server-VLAN |
| /25 | 255.255.255.128 | 128 | 126 | |
| /26 | 255.255.255.192 | 64 | 62 | |
| /27 | 255.255.255.224 | 32 | 30 | |
| /28 | 255.255.255.240 | 16 | 14 | kleine Kunden-DMZ |
| /29 | 255.255.255.248 | 8 | 6 | Kundennetz mit wenigen öffentl. IPs |
| /30 | 255.255.255.252 | 4 | 2 | Transfernetz (alt) |
| /31 | 255.255.255.254 | 2 | 2 (RFC 3021) | Point-to-Point im RZ / Spine-Leaf |
| /32 | 255.255.255.255 | 1 | 1 | Loopback, Host-Route |

- [ ] Methode „magische Zahl“: Blockgröße = 256 − Maskenwert im interessanten Oktett
- [ ] Für eine IP + Präfix bestimmen: **Netz-ID, erster Host, letzter Host, Broadcast, Anzahl Hosts**
- [ ] Ein Netz in n gleiche Subnetze teilen; VLSM (größte Netze zuerst vergeben)
- [ ] Summarization: 10.1.0.0/24 – 10.1.3.0/24 → 10.1.0.0/22
- [ ] Prüfen, ob zwei IPs im selben Netz sind

**Übungsaufgaben (Lösung unten):**
1. 172.16.45.130/27 → Netz, Broadcast, Hostbereich?
2. 10.20.0.0/16 soll in Subnetze mit je max. 500 Hosts geteilt werden – welches Präfix, wie viele Netze?
3. Fasse 192.168.8.0/24 bis 192.168.15.0/24 zusammen.
4. Sind 10.0.5.200/23 und 10.0.4.10/23 im selben Netz?

<details><summary>Lösungen</summary>

1. Block 32 → Netz 172.16.45.128, Broadcast .159, Hosts .129–.158
2. /23 (510 Hosts), 2^(23−16) = 128 Netze
3. 192.168.8.0/21
4. Ja – /23 = Block 2 im 3. Oktett → 10.0.4.0–10.0.5.255
</details>

Trainieren: subnettingpractice.com, subnetipv4.com – **täglich 10 Aufgaben, bis du unter 30 Sekunden pro Aufgabe bist.**

## 3. IPv6

- [ ] 128 Bit, 8 Blöcke à 16 Bit hex; Kürzung: führende Nullen weg, **einmal** `::` für aufeinanderfolgende Null-Blöcke
- [ ] Präfix-Konventionen: /64 pro LAN (SLAAC braucht /64!), /56 oder /48 pro Standort/Kunde, /127 für Point-to-Point
- [ ] Adresstypen:

| Typ | Präfix | Bemerkung |
|---|---|---|
| Global Unicast | 2000::/3 | öffentlich routbar |
| Unique Local (ULA) | fc00::/7 (praktisch fd00::/8) | „privat“ |
| Link-Local | fe80::/10 | auf jedem Interface, nicht geroutet |
| Multicast | ff00::/8 | ff02::1 alle Knoten, ff02::2 alle Router |
| Loopback | ::1/128 | |
| Unspecified | ::/128 | |
| Dokumentation | 2001:db8::/32 | |

- [ ] **Kein Broadcast** in IPv6 – stattdessen Multicast
- [ ] **NDP** (Neighbor Discovery, ICMPv6) ersetzt ARP: Neighbor Solicitation/Advertisement, Router Solicitation/Advertisement
- [ ] Adressvergabe: SLAAC, EUI-64, Privacy Extensions, DHCPv6 (stateful/stateless), M-/O-Flags im RA
- [ ] Dual Stack, Tunnel (6in4), NAT64/DNS64 (Grundzüge)
- [ ] **ICMPv6 nicht pauschal blocken** – sonst bricht NDP und Path-MTU-Discovery

## 4. Transportschicht

- [ ] **TCP**: verbindungsorientiert, zuverlässig, Reihenfolge, Flusskontrolle
  - 3-Way-Handshake: SYN → SYN/ACK → ACK
  - Abbau: FIN/ACK (4-Way) oder RST (hart)
  - Flags: SYN, ACK, FIN, RST, PSH, URG
  - Sequenz-/Bestätigungsnummern, Window Size, Retransmissions, MSS
  - Zustände: LISTEN, SYN_SENT, SYN_RECEIVED, ESTABLISHED, FIN_WAIT, **TIME_WAIT**, CLOSE_WAIT (→ siehe [`tools/netstat-ss.md`](../tools/netstat-ss.md))
- [ ] **UDP**: verbindungslos, kein Handshake, geringer Overhead (DNS, DHCP, NTP, SNMP, Syslog, VoIP, VXLAN)
- [ ] Ports: Well-known 0–1023, Registered 1024–49151, Dynamic/Ephemeral 49152–65535
- [ ] Socket = IP + Port + Protokoll

### Ports – auswendig lernen

| Port | Protokoll | Dienst |
|---|---|---|
| 20/21 | TCP | FTP (Daten/Steuerung) |
| 22 | TCP | SSH, SCP, SFTP |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 49 | TCP | TACACS+ |
| 53 | UDP/TCP | DNS (TCP bei großen Antworten / Zonentransfer) |
| 67/68 | UDP | DHCP Server/Client |
| 69 | UDP | TFTP |
| 80 | TCP | HTTP |
| 88 | TCP/UDP | Kerberos |
| 110 / 995 | TCP | POP3 / POP3S |
| 123 | UDP | NTP |
| 135 | TCP | MS RPC |
| 137–139 | UDP/TCP | NetBIOS |
| 143 / 993 | TCP | IMAP / IMAPS |
| 161 / 162 | UDP | SNMP / SNMP-Trap |
| 179 | TCP | BGP |
| 389 / 636 | TCP | LDAP / LDAPS |
| 443 | TCP (UDP bei QUIC/HTTP3) | HTTPS |
| 445 | TCP | SMB |
| 465 / 587 | TCP | SMTPS / Submission |
| 500 / 4500 | UDP | IKE / IPsec NAT-T |
| 514 | UDP | Syslog |
| 873 | TCP | rsync |
| 1433 | TCP | MS SQL |
| 1812/1813 | UDP | RADIUS |
| 2049 | TCP/UDP | NFS |
| 3260 | TCP | iSCSI |
| 3306 | TCP | MySQL/MariaDB |
| 3389 | TCP | RDP |
| 4789 | UDP | VXLAN |
| 5432 | TCP | PostgreSQL |
| 5666 | TCP | NRPE (Nagios) |
| 5985/5986 | TCP | WinRM HTTP/HTTPS |
| 6443 | TCP | Kubernetes API |
| 8080 / 8443 | TCP | alternatives HTTP(S) |
| 9100 | TCP | Prometheus node_exporter |
| 10050/10051 | TCP | Zabbix Agent/Server |

## 5. Hilfsprotokolle

- [ ] **ARP**: Request (Broadcast „Wer hat 10.0.0.1?“), Reply (Unicast), ARP-Cache, Gratuitous ARP (wichtig bei Failover von VIPs/VMs!), Proxy-ARP
- [ ] **ICMP**: Typ 0 Echo Reply, 3 Destination Unreachable (Code 4 = Fragmentation needed → PMTUD), 5 Redirect, 8 Echo Request, 11 Time Exceeded (→ traceroute)
- [ ] Wie `ping` und `traceroute` technisch funktionieren (TTL!)

## 6. Befehle dieser Phase

### Linux
```bash
ip -br a                         # alle IPs kurz
ip -br link                      # Linkstatus
ip addr add 10.0.0.10/24 dev eth0
ip route                         # Routing-Tabelle
ip route get 8.8.8.8             # welche Route/Quell-IP wird genutzt?
ip neigh                         # ARP-/NDP-Cache
ip -6 a ; ip -6 route
ethtool eth0                     # Speed, Duplex, Link
ethtool -S eth0                  # Fehlerzähler der NIC
ethtool -m eth0                  # Transceiver-Werte (wenn unterstützt)
ping -c 4 10.0.0.1
ping -M do -s 1472 10.0.0.1      # MTU 1500 testen (1472 + 28 Header)
ping -M do -s 8972 10.0.0.1      # Jumbo 9000 testen
traceroute -n 8.8.8.8 ; mtr -n 8.8.8.8
ss -tulpn                        # offene Ports
arping -I eth0 10.0.0.1          # ARP-Ebene testen (findet doppelte IPs)
```

### Windows
```powershell
ipconfig /all
ipconfig /release ; ipconfig /renew
arp -a
route print
netstat -ano
ping -n 4 -l 1472 -f 10.0.0.1     # -f = Don't Fragment
tracert -d 8.8.8.8
pathping 8.8.8.8
Get-NetIPConfiguration
Get-NetAdapter | Format-Table Name, Status, LinkSpeed
Get-NetNeighbor
Test-NetConnection 10.0.0.5 -Port 443
```

### Cisco
```
show ip interface brief
show interfaces Gi1/0/1              # Fehlerzähler, Duplex, MTU
show interfaces counters errors
show arp
show ipv6 interface brief
show ipv6 neighbors
ping 10.0.0.1 size 1500 df-bit
traceroute 10.0.0.1
```

## 7. Praxisaufgaben

- [ ] Wireshark: einen kompletten HTTP-Aufruf mitschneiden und erklären (DNS → ARP → TCP-Handshake → HTTP → FIN)
- [ ] Eigene Rechnung: das IP-Konzept für ein fiktives Kunden-Rack (Management /27, Produktion /25, Storage /26, Transfernetze /31) entwerfen
- [ ] Im Betrieb: `show interfaces` auf einem echten Switch lesen und Fehlerzähler interpretieren (mit Ausbilder)
- [ ] MTU-Test zwischen zwei Servern durchführen (mit und ohne Jumbo Frames)

## Meine Notizen

