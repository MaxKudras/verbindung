# Netzwerk Phase 5 – Netzwerksicherheit

**Dauer:** 4–6 Wochen · **Voraussetzung:** Phasen 1–4

---

## 1. Grundprinzipien

- [ ] Schutzziele: Vertraulichkeit, Integrität, Verfügbarkeit (+ Authentizität, Nichtabstreitbarkeit)
- [ ] **Defense in Depth**, **Least Privilege**, **Zero Trust**, Segmentierung/Mikrosegmentierung
- [ ] Zonenkonzept: Internet → DMZ → intern → Management → Backup (Backup-Netz besonders schützen!)
- [ ] Default Deny

## 2. Firewalls

- [ ] Paketfilter (stateless) vs. **Stateful Inspection** vs. Application Layer / **NGFW** (App-ID, User-ID, IPS, URL-Filter, TLS-Inspection)
- [ ] Regelaufbau: Quelle, Ziel, Dienst/Port, Aktion, Logging; Reihenfolge „first match“
- [ ] Objekte & Gruppen statt einzelner IPs
- [ ] Regel-Hygiene: Kommentare, Ticketnummer, Ablaufdatum, ungenutzte Regeln aufräumen (Hit Counter)
- [ ] HA: Active/Passive, Active/Active, Session-Sync
- [ ] Hersteller: Fortinet (FortiGate), Palo Alto, Check Point, Cisco FTD/ASA, Sophos, Juniper SRX, OPNsense/pfSense
- [ ] FortiGate-CLI-Basics: `get system status`, `diagnose sniffer packet any 'host 10.0.0.5' 4`, `diagnose debug flow` – je nach eingesetztem Hersteller vertiefen

## 3. ACLs auf Routern/Switches

- [ ] Standard-ACL (nur Quelle, nah am Ziel) vs. Extended (Quelle, Ziel, Protokoll, Port – nah an der Quelle)
- [ ] Wildcard-Masken (invertierte Maske: /24 → 0.0.0.255)
- [ ] **Implizites `deny any` am Ende**
- [ ] Inbound vs. Outbound
- [ ] VTY-ACL für Management-Zugriff, Control-Plane-Policing (CoPP)
```
ip access-list extended MGMT-IN
 permit tcp 10.99.0.0 0.0.0.255 any eq 22
 deny   ip any any log
line vty 0 15
 access-class MGMT-IN in
 transport input ssh
show access-lists
```

## 4. Linux-Firewall

- [ ] Netfilter-Konzept: Tabellen (filter, nat, mangle), Chains (INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING)
- [ ] **nftables** (aktuell), iptables (Legacy, noch weit verbreitet), Frontends firewalld (RHEL) und ufw (Ubuntu)
```bash
nft list ruleset
iptables -L -n -v --line-numbers
iptables -t nat -L -n -v
firewall-cmd --get-active-zones
firewall-cmd --list-all
firewall-cmd --permanent --add-service=https && firewall-cmd --reload
ufw status verbose
ufw allow from 10.99.0.0/24 to any port 22 proto tcp
```

## 5. VPN

- [ ] **IPsec**:
  - IKEv1 (Main/Aggressive Mode) vs. **IKEv2**
  - Phase 1 (IKE SA: Verschlüsselung, Hash, DH-Gruppe, Lifetime, Auth PSK/Zertifikat)
  - Phase 2 (IPsec SA / Child SA, Proxy-IDs / Traffic Selectors – **müssen auf beiden Seiten spiegelbildlich passen**)
  - ESP (IP 50) vs. AH (IP 51), Tunnel- vs. Transport-Mode, NAT-T (UDP 4500), PFS
  - Policy-based vs. Route-based (VTI)
- [ ] SSL-/TLS-VPN (Remote Access), **WireGuard** (modern, schlank), OpenVPN
- [ ] Typische Fehler: Proposal-Mismatch, falsche Subnetze in Phase 2, Firewall blockt UDP 500/4500, abgelaufenes Zertifikat
- [ ] Linux strongSwan: `swanctl --list-sas`, `ipsec statusall` (Legacy)

## 6. Kryptografie-Grundlagen

- [ ] Symmetrisch (AES) vs. asymmetrisch (RSA, ECC), Hybride Verfahren
- [ ] Hashes (SHA-256), HMAC, digitale Signatur
- [ ] **PKI**: CA, Zwischenzertifikate, Zertifikatskette, CSR, CRL/OCSP, Ablaufüberwachung!
- [ ] TLS-Handshake (Grundzüge TLS 1.2 vs 1.3)
```bash
openssl s_client -connect www.example.com:443 -servername www.example.com
openssl x509 -in cert.pem -noout -subject -issuer -dates
openssl req -new -newkey rsa:4096 -nodes -keyout srv.key -out srv.csr
```

## 7. Angriffe kennen (und Gegenmaßnahmen)

| Angriff | Schicht | Gegenmaßnahme |
|---|---|---|
| MAC-Flooding | L2 | Port Security |
| ARP-Spoofing / MITM | L2 | Dynamic ARP Inspection |
| Rogue DHCP | L2/L7 | DHCP Snooping |
| VLAN-Hopping (Switch Spoofing, Double Tagging) | L2 | DTP aus, Native VLAN ungenutzt |
| STP-Manipulation | L2 | BPDU Guard, Root Guard |
| IP-Spoofing | L3 | uRPF, BCP38-Filter |
| (D)DoS / SYN-Flood | L3/L4 | Scrubbing, RTBH (BGP-Blackhole), SYN-Cookies, Rate Limiting |
| BGP-Hijacking | L3 | RPKI, Prefix-Filter |
| DNS-Spoofing / Amplification | L7 | DNSSEC, kein offener Resolver |
| Brute Force auf SSH | L7 | Key-Auth, fail2ban, ACL |

## 8. Härtung von Netzwerkgeräten

- [ ] SSH v2 statt Telnet, HTTPS statt HTTP, SNMPv3 statt v2c
- [ ] Starke Passwörter, `service password-encryption`, `enable secret` (nicht `enable password`)
- [ ] AAA (TACACS+) mit lokalem Fallback
- [ ] Management nur aus Management-VRF/-Netz bzw. OOB
- [ ] Ungenutzte Dienste aus (`no ip http server`, `no cdp run` an Außenports)
- [ ] Login-Banner, Exec-Timeout, Logging an zentralen Server, NTP
- [ ] Firmware aktuell (Hersteller-Advisories abonnieren, CVEs prüfen)
- [ ] Konfig-Backups (→ Phase 6, Oxidized + Git)

## 9. Analyse & Prüfung

- [ ] Paketmitschnitte → [`tools/tcpdump-wireshark.md`](../tools/tcpdump-wireshark.md)
- [ ] SPAN / RSPAN / ERSPAN (Port-Mirroring auf Switch), TAPs
- [ ] IDS/IPS: Suricata, Snort, Zeek
- [ ] Port-Scans **nur mit Freigabe**:
```bash
nmap -sn 10.0.0.0/24                 # Ping-Sweep
nmap -sS -p 1-1024 10.0.0.5          # SYN-Scan
nmap -sV -O 10.0.0.5                 # Dienst- und OS-Erkennung
nmap -sU -p 53,123,161 10.0.0.5      # UDP
nc -zv 10.0.0.5 22                   # einzelnen Port testen
```
- [ ] Schwachstellen-Scanner: Greenbone/OpenVAS, Nessus
- [ ] SIEM-Grundzüge (Splunk, Elastic, Wazuh)

## 10. Praxisaufgaben

- [ ] OPNsense im Labor: WAN, LAN, DMZ; Webserver in DMZ nur per 443 erreichbar
- [ ] Site-to-Site-IPsec (IKEv2) zwischen zwei OPNsense/strongSwan; danach absichtlich Proposal ändern und Logs lesen
- [ ] WireGuard-Tunnel zwischen zwei Linux-VMs
- [ ] Switch im Labor nach Checkliste aus Abschnitt 8 härten
- [ ] Im Betrieb: Firewall-Change begleiten – wie wird eine Regel beantragt, geprüft, umgesetzt, dokumentiert?

## Meine Notizen

