# Heimlabor & Praxisprojekte

> Lesen bringt dich 20 % weit, **selbst bauen und kaputtmachen** die restlichen 80 %. Im Labor darfst du Fehler machen, die im RZ teuer wären.

## 1. Hardware-Optionen

| Option | Kosten | Geeignet für |
|---|---|---|
| Eigener PC/Laptop mit 32 GB RAM + VirtualBox/VMware Workstation | 0 € | Einstieg, einzelne VMs, Packet Tracer |
| Gebrauchter Mini-PC (Lenovo Tiny, HP Mini, Dell Micro) mit 32–64 GB RAM, NVMe | 150–400 € | **Empfehlung**: Proxmox, 10–20 VMs, leise, stromsparend |
| 3 Mini-PCs | 450–900 € | echtes Cluster (Proxmox HA, Ceph, k3s) |
| Gebrauchter Server (Dell R630/R730, HPE DL360/380) | 200–600 € | iDRAC/iLO üben, viel RAM – aber laut und stromhungrig |
| Gebrauchter Managed Switch (Cisco Catalyst, Aruba, MikroTik) | 30–150 € | echte VLANs, Trunks, LACP, CLI |

Zusätzlich vielleicht im Betrieb verfügbar: ausgemusterte Hardware, Labor-Racks – **frag deinen Ausbilder!**

## 2. Software (kostenlos)

| Bereich | Software |
|---|---|
| Hypervisor | **Proxmox VE**, XCP-ng, ESXi (Evaluierung), Hyper-V (in Windows Pro) |
| Netzwerksimulation | **Cisco Packet Tracer** (CCNA), **GNS3**, **EVE-NG** Community, **Containerlab** (cEOS, FRRouting, Nokia SR Linux) |
| Firewall | **OPNsense**, pfSense CE |
| Linux | Rocky Linux, AlmaLinux, Ubuntu Server, Debian |
| Windows | Windows Server Evaluierung (180 Tage, verlängerbar/neu aufsetzbar) |
| Monitoring | Checkmk Raw, Icinga 2, Zabbix, Prometheus + Grafana, LibreNMS |
| Doku/IPAM | NetBox (Docker), draw.io |
| Backup | Proxmox Backup Server, Veeam Community Edition, Restic, Bareos |
| Storage | TrueNAS, Ceph (in Proxmox) |
| Automatisierung | Ansible, Terraform/OpenTofu, Gitea/GitLab CE |

## 3. Referenz-Labornetz

```
                     Internet (Heimrouter)
                            │
                     ┌──────┴──────┐
                     │  OPNsense   │  WAN
                     └──┬───┬───┬──┘
          VLAN 10 MGMT  │   │   │  VLAN 30 DMZ
         10.10.10.0/24  │   │   │  10.10.30.0/24
                        │   │ VLAN 20 SERVER
                        │   │ 10.10.20.0/24
                  ┌─────┴───┴───┴─────┐
                  │ Proxmox (vmbr0,   │
                  │  VLAN-aware)      │
                  └───────────────────┘
  VMs: dc01, dc02 (Windows AD) · srv-web01/02 (Rocky/Ubuntu) · db01 · mon01 (Monitoring)
       ansible01 (Control Node + Git) · backup01 (PBS) · netbox01 · k3s-1..3
  VLAN 40 STORAGE 10.10.40.0/24 (NFS/iSCSI, MTU 9000) · VLAN 99 OOB
```

## 4. Projekte

Jedes Projekt hat **Abnahmekriterien**. Erst wenn alle erfüllt sind und alles dokumentiert ist (in diesem Repo unter `labor/projekte/<name>.md`), gilt es als fertig.

### Projekt 1 – Fundament: Proxmox + OPNsense + VLANs
- [ ] Proxmox installiert, VLAN-aware Bridge, OPNsense als VM mit VLANs 10/20/30/40
- [ ] Firewall-Regeln: MGMT darf alles, SERVER darf ins Internet, DMZ darf nur DNS/NTP/HTTP(S) raus, nichts von DMZ nach SERVER
- [ ] **Abnahme:** Netzplan (draw.io) + Regelwerk dokumentiert; Test mit `nc`/`curl` zeigt erlaubte/verbotene Verbindungen

### Projekt 2 – Netzwerk-Lab: Switching & Routing (GNS3/Containerlab)
- [ ] 4 Switches, VLANs, Trunks, Rapid-PVST mit definierter Root, LACP
- [ ] 3 Router mit OSPF (2 Areas), 1 Router mit eBGP zu „Provider“
- [ ] HSRP/VRRP für Gateway-Redundanz
- [ ] **Abnahme:** Link-/Router-Ausfall simuliert, Konvergenz dokumentiert, alle `show`-Ausgaben erklärt

### Projekt 3 – Spine-Leaf mit VXLAN/EVPN (Fortgeschritten, Containerlab)
- [ ] 2 Spines, 4 Leafs (Arista cEOS oder FRR), eBGP-Underlay, ECMP
- [ ] EVPN-Overlay mit L2VNI (VLAN 100 auf allen Leafs) und L3VNI (Routing zwischen 2 VRFs)
- [ ] **Abnahme:** Hosts auf Leaf1 und Leaf4 im selben Subnetz pingen sich; `show bgp evpn` erklärt

### Projekt 4 – Active Directory
- [ ] 2 DCs, DNS, DHCP-Failover, OU-Struktur, 50 Benutzer per PowerShell/CSV
- [ ] Fileserver mit AGDLP, 3 GPOs, Windows LAPS
- [ ] **Abnahme:** DC01 aus → Anmeldung funktioniert weiter; `dcdiag` / `repadmin` fehlerfrei

### Projekt 5 – Linux-Webplattform
- [ ] 2 Webserver (Nginx) + HAProxy/Nginx-Loadbalancer + MariaDB/PostgreSQL
- [ ] TLS (eigene CA oder Let's Encrypt), SELinux enforcing, Firewall, fail2ban
- [ ] **Abnahme:** ein Webserver aus → Seite bleibt erreichbar; Lynis-Score dokumentiert

### Projekt 6 – Automatisierung mit Ansible + Git
- [ ] Gitea/GitLab-Repo, Ansible-Rolle „baseline“ für alle Linux-Server, Rolle für Nginx
- [ ] Netzwerk-Playbook für die Labor-Switches (NTP, Syslog, SNMPv3, Banner)
- [ ] **Abnahme:** neuer Server ist nach `ansible-playbook` vollständig konfiguriert; zweiter Lauf = 0 changed

### Projekt 7 – Monitoring & Logging
- [ ] Checkmk/Icinga **oder** Zabbix für Hosts/Dienste/Switches; Prometheus + Grafana für Metriken
- [ ] Zentrales Logging (Graylog oder Loki), alle Server und die Firewall senden Logs
- [ ] Eigenes Plugin, Abhängigkeiten, Alarm per E-Mail/Teams/Telegram
- [ ] **Abnahme:** Dienst stoppen → Alarm innerhalb von 5 min; Switch aus → nur ein Alarm

### Projekt 8 – Backup & Restore
- [ ] Backup-Konzept (RPO/RTO je System, 3-2-1-1-0) schriftlich
- [ ] Proxmox Backup Server für VMs, Restic für Konfigurationen, DB-Dumps
- [ ] **Abnahme:** dokumentierter Restore-Test: VM, einzelne Datei, Datenbank, AD-Objekt

### Projekt 9 – Storage & Cluster
- [ ] 3-Knoten-Proxmox-Cluster (zur Not verschachtelt), Ceph oder NFS-Shared-Storage, HA
- [ ] Storage-VLAN mit MTU 9000, Multipath zu iSCSI-Target
- [ ] **Abnahme:** Knoten hart ausschalten → VM startet automatisch woanders

### Projekt 10 – Container & Kubernetes
- [ ] Docker-Compose-Stack, dann k3s-Cluster mit 3 Knoten, Ingress, persistente Volumes
- [ ] **Abnahme:** App-Update per Rolling Update ohne Ausfall, Rollback demonstriert

### Projekt 11 – Source of Truth & IaC
- [ ] NetBox mit allen Labor-Geräten, IPs, VLANs, Kabeln
- [ ] Terraform legt VMs in Proxmox an, Ansible konfiguriert sie, Inventory kommt aus NetBox
- [ ] **Abnahme:** Neue VM = Eintrag in NetBox + `terraform apply` + `ansible-playbook`

### Projekt 12 – Härtung & Security
- [ ] VPN (WireGuard + IPsec Site-to-Site), Greenbone-Scan des Labors, Findings beheben
- [ ] Suricata auf OPNsense, Zertifikatsüberwachung
- [ ] **Abnahme:** Scan-Report vorher/nachher

## 5. Dokumentationsvorlage für Projekte

```markdown
# Projekt X – Titel
## Ziel
## Netzplan / Architektur
## Umsetzung (Schritte + wichtige Befehle)
## Tests & Ergebnisse
## Probleme und Lösungen
## Was ich gelernt habe
## Offene Punkte
```
