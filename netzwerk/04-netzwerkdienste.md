# Netzwerk Phase 4 – Netzwerkdienste

**Dauer:** 3–4 Wochen · **Voraussetzung:** Phasen 1–3

---

## 1. DNS – Pflicht, mit `dig`

> Tiefgang zu `dig`, `nslookup`, `host`, `Resolve-DnsName` → [`tools/dig-dns.md`](../tools/dig-dns.md)

- [ ] Hierarchie: Root (.) → TLD (.de) → Second Level (firma.de) → Subdomain
- [ ] **Rekursiver Resolver** vs. **autoritativer Server**; iterative vs. rekursive Anfrage
- [ ] Caching, **TTL**, Negative Caching (SOA-Minimum)
- [ ] Record-Typen:

| Typ | Zweck | Beispiel |
|---|---|---|
| A | Name → IPv4 | `www IN A 203.0.113.10` |
| AAAA | Name → IPv6 | |
| CNAME | Alias (nicht am Zonen-Apex!) | `web IN CNAME www` |
| MX | Mailserver mit Priorität | `@ IN MX 10 mail` |
| NS | zuständige Nameserver | |
| SOA | Zonen-Metadaten (Serial, Refresh, Retry, Expire, Minimum) | |
| PTR | Reverse: IP → Name | `10.113.0.203.in-addr.arpa` |
| TXT | SPF, DKIM, DMARC, Verifikation | |
| SRV | Dienst-Lokalisierung (AD!) | `_ldap._tcp.dc._msdcs.firma.local` |
| CAA | erlaubte Zertifizierungsstellen | |

- [ ] Zonen, Zonentransfer (AXFR/IXFR, TCP 53), Primary/Secondary, Serial-Nummer erhöhen!
- [ ] Reverse-Zonen (in-addr.arpa, ip6.arpa)
- [ ] Split-Horizon-DNS (intern andere Antwort als extern)
- [ ] DNSSEC-Grundzüge (RRSIG, DNSKEY, DS, Chain of Trust)
- [ ] DNS over TLS/HTTPS (Grundzüge)
- [ ] Software: BIND, Unbound, PowerDNS, Windows DNS, dnsmasq
- [ ] Typische Fehler: falsche TTL vor Migration, vergessene Serial-Erhöhung, fehlender PTR (Mailprobleme!), CNAME-Konflikte

## 2. DHCP

- [ ] **DORA**: Discover (Broadcast) → Offer → Request → Acknowledge
- [ ] Lease, T1 (50 %) / T2 (87,5 %) Renewal
- [ ] Scopes, Ausschlüsse, Reservierungen (MAC-gebunden)
- [ ] Wichtige Optionen: 1 Subnetzmaske, 3 Router, 6 DNS, 15 Domain, 42 NTP, 51 Lease, **66/67 TFTP-Server/Bootfile (PXE)**, 82 Relay Agent Info
- [ ] **DHCP-Relay** (`ip helper-address`) – Broadcast über Router hinweg
- [ ] Failover / Hochverfügbarkeit (ISC/Kea, Windows DHCP-Failover)
- [ ] DHCPv6 + SLAAC (Phase 1)
- [ ] Sicherheit: Rogue DHCP → DHCP Snooping (Phase 2)

## 3. NTP & Zeit

- [ ] Warum kritisch: Logs korrelieren, **Kerberos** (max. 5 min Abweichung), TLS-Zertifikate, Cluster, Datenbanken
- [ ] Stratum 0 (Atomuhr/GPS) → Stratum 1 → Stratum 2 …
- [ ] Mindestens 3–4 Zeitquellen, interne NTP-Server, PTP (Precision Time Protocol) für Hochfrequenz-Anwendungen
- [ ] Befehle:
```bash
chronyc sources -v ; chronyc tracking
timedatectl
```
```powershell
w32tm /query /status
w32tm /query /peers
w32tm /resync
```
```
show ntp status
show ntp associations
```

## 4. AAA – Authentifizierung für Netzwerkgeräte

- [ ] Authentication, Authorization, Accounting
- [ ] **TACACS+** (TCP 49, Cisco, verschlüsselt komplettes Paket, Befehls-Autorisierung) vs. **RADIUS** (UDP 1812/1813, nur Passwort verschlüsselt, für 802.1X/VPN)
- [ ] Lokaler Fallback-Account bei AAA-Ausfall
- [ ] Software: Cisco ISE, FreeRADIUS, tac_plus, Windows NPS
- [ ] Rollenbasierter Zugriff (Privilege Levels, Read-only-Accounts für Monitoring)

## 5. SNMP

- [ ] Manager / Agent, **MIB**, **OID** (z. B. `1.3.6.1.2.1.1.5.0` = sysName)
- [ ] Operationen: Get, GetNext, GetBulk, Set, **Trap**, Inform
- [ ] Versionen: v1, **v2c** (Community = Klartext-Passwort!), **v3** (Auth: SHA, Priv: AES – **Standard verwenden**)
- [ ] Wichtige MIBs: IF-MIB (ifInOctets, ifHCInOctets für 64-Bit-Zähler!), HOST-RESOURCES-MIB, ENTITY-MIB
- [ ] Befehle:
```bash
snmpwalk -v2c -c public 10.0.0.1 1.3.6.1.2.1.1
snmpget  -v3 -l authPriv -u monitor -a SHA -A 'authpass' -x AES -X 'privpass' 10.0.0.1 sysName.0
snmpwalk -v2c -c public 10.0.0.1 IF-MIB::ifDescr
```

## 6. Syslog

- [ ] Facilities (local0–7), **Severity 0–7** – Merksatz: *„Every Awesome Cisco Engineer Will Need Ice cream Daily“*

| Level | Name |
|---|---|
| 0 | Emergency |
| 1 | Alert |
| 2 | Critical |
| 3 | Error |
| 4 | Warning |
| 5 | Notice |
| 6 | Informational |
| 7 | Debug |

- [ ] Zentraler Syslog-Server (rsyslog, syslog-ng, Graylog), UDP 514 / TCP 514 / TLS 6514
- [ ] Zeitstempel + NTP auf allen Geräten
- [ ] `logger -p local0.warn "Test"` (Linux-Testnachricht)

## 7. Flow-Daten

- [ ] **NetFlow** (Cisco) / **IPFIX** (Standard) / **sFlow** (Sampling) – „Wer redet mit wem, wie viel?“
- [ ] Einsatz: Top-Talker finden, DDoS-Erkennung, Kapazitätsplanung
- [ ] Tools: ntopng, Kentik, ElastiFlow, pmacct

## 8. Weitere Dienste im RZ

- [ ] **Load Balancer**: L4 vs. L7, VIP, Pool, Health Checks, Algorithmen (Round Robin, Least Connections), SSL-Offloading, Persistence – F5 BIG-IP, HAProxy, NGINX, Citrix ADC
- [ ] **PXE-Boot**: DHCP (66/67) + TFTP/HTTP → automatisierte Installation
- [ ] **IPAM / DCIM**: NetBox, phpIPAM, Infoblox – *Wo ist welche IP, welches Gerät in welchem Rack?*
- [ ] **Konsolen-Server** (Opengear, Lantronix) für seriellen Zugriff bei Netzausfall
- [ ] QoS-Grundzüge (DSCP, Klassifizierung, Queuing)

## 9. Praxisaufgaben

- [ ] BIND oder Unbound auf Linux aufsetzen: eigene Zone `labor.lan` mit A, CNAME, MX, PTR; mit `dig` testen
- [ ] DHCP-Server (Kea oder isc-dhcp) + Relay auf einem Router im Labor
- [ ] SNMPv3 auf einem Switch/Linux einrichten und mit `snmpwalk` abfragen
- [ ] Alle Laborgeräte loggen an einen zentralen rsyslog/Graylog
- [ ] Im Betrieb: herausfinden, welche DNS-, NTP-, Syslog- und AAA-Server euer RZ nutzt (nur Rollen notieren, keine IPs ins Repo!)

## Meine Notizen

