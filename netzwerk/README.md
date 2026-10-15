# Lernpfad Netzwerk

Ziel: Am Ende der Ausbildung ein Netzwerk im Rechenzentrum **selbständig betreiben, erweitern und Störungen strukturiert lösen** können – auf dem Niveau eines CCNA-zertifizierten Junior-Netzwerkadministrators, mit Ausblick auf CCNP/DC.

## Phasen

| Phase | Datei | Dauer | Kernthemen |
|---|---|---|---|
| 1 | [01-grundlagen-adressierung.md](01-grundlagen-adressierung.md) | 4–6 Wo. | Ethernet, IPv4/Subnetting, IPv6, TCP/UDP, Ports, ARP, ICMP |
| 2 | [02-switching.md](02-switching.md) | 4–6 Wo. | VLAN, Trunk, STP, LACP, MLAG, Spine-Leaf, Switch-Security |
| 3 | [03-routing.md](03-routing.md) | 6–8 Wo. | Statisch, OSPF, BGP, FHRP, VRF, NAT, VXLAN/EVPN |
| 4 | [04-netzwerkdienste.md](04-netzwerkdienste.md) | 3–4 Wo. | DNS (dig!), DHCP, NTP, AAA, SNMP, Syslog, Flow |
| 5 | [05-sicherheit.md](05-sicherheit.md) | 4–6 Wo. | Firewalls, ACL, VPN/IPsec, Angriffe, Härtung, Mitschnitte |
| 6 | [06-monitoring-automatisierung.md](06-monitoring-automatisierung.md) | laufend | Monitoring, NetBox, Git, Ansible, Python, Troubleshooting-Methodik |

## Werkzeuge, die dich durch den gesamten Pfad begleiten

- [`tools/netstat-ss.md`](../tools/netstat-ss.md) – Verbindungen & offene Ports
- [`tools/dig-dns.md`](../tools/dig-dns.md) – DNS-Diagnose
- [`tools/ping-traceroute-mtr.md`](../tools/ping-traceroute-mtr.md) – Erreichbarkeit & Pfad
- [`tools/tcpdump-wireshark.md`](../tools/tcpdump-wireshark.md) – Paketmitschnitte
- [`tools/ssh.md`](../tools/ssh.md) – Fernzugriff, Jump-Hosts, Tunnel
- [`tools/git.md`](../tools/git.md) – Konfigurationen versionieren
- [`tools/ansible.md`](../tools/ansible.md) – Netzwerkautomatisierung
- [`tools/monitoring-nagios.md`](../tools/monitoring-nagios.md) – Nagios, Icinga, Zabbix, Prometheus

## Hersteller-Hinweis

Die Beispiele nutzen **Cisco IOS/IOS-XE** (Prüfungs- und Lehrbuchstandard). Im RZ triffst du oft auch:

| Hersteller / OS | Ähnlich zu | Besonderheit |
|---|---|---|
| Cisco NX-OS (Nexus) | IOS | `feature`-Befehle, vPC, im RZ sehr verbreitet |
| Arista EOS | IOS (fast identisch) | Linux-Basis, `bash` direkt auf dem Switch, eAPI |
| Juniper Junos | eigene Syntax | Hierarchisch, `commit` / `rollback`, `show | display set` |
| Nokia SR Linux / SONiC / Cumulus | Linux | Open Networking |
| HPE Aruba CX | IOS-ähnlich | |
| Fortinet / Palo Alto | – | Firewalls, eigene CLI + GUI |

**Aufgabe:** Finde heraus, welche Hersteller in deinem RZ im Einsatz sind, und notiere es hier:

- Core/Spine: …
- Leaf/ToR: …
- Firewall: …
- Load Balancer: …
- Management/OOB-Switches: …

## Vergleich Cisco ↔ Juniper ↔ Arista (Spickzettel)

| Aufgabe | Cisco IOS | Juniper Junos | Arista EOS |
|---|---|---|---|
| Konfig anzeigen | `show running-config` | `show configuration` | `show running-config` |
| Interfaces kurz | `show ip interface brief` | `show interfaces terse` | `show ip interface brief` |
| Konfig-Modus | `configure terminal` | `configure` | `configure` |
| Speichern/Aktivieren | `write memory` | `commit` | `write memory` |
| Änderung zurücknehmen | (manuell / `configure replace`) | `rollback 1` + `commit` | `configure session` + `abort` |
| Routing-Tabelle | `show ip route` | `show route` | `show ip route` |
| Nachbarn | `show lldp neighbors` | `show lldp neighbors` | `show lldp neighbors` |
| MAC-Tabelle | `show mac address-table` | `show ethernet-switching table` | `show mac address-table` |
| Logs | `show logging` | `show log messages` | `show logging` |
| Filter | `| include`, `| section` | `| match`, `| except` | `| include`, `| section` |

## Abschlussziele des Netzwerk-Pfads

- [ ] Ich kann einen neuen Server-Port auf einem Leaf-Switch korrekt konfigurieren (VLAN, Beschreibung, LACP), dokumentieren und testen.
- [ ] Ich kann ein neues Kunden-VLAN/-Netz end-to-end anlegen (IPAM → Switch → Gateway → Firewall-Regel → Monitoring).
- [ ] Ich kann einen „Server nicht erreichbar“-Incident strukturiert von L1 bis L7 eingrenzen und die Ursache benennen.
- [ ] Ich kann eine BGP-Session zu einem Upstream/Peer prüfen und erklären, welche Präfixe gelernt/announced werden.
- [ ] Ich habe mindestens ein wiederkehrendes Netzwerk-Task mit Ansible oder Python automatisiert.
- [ ] Ich kenne das OOB-Management-Netz meines RZ und komme bei Ausfall des Produktionsnetzes trotzdem auf die Geräte.
