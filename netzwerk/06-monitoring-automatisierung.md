# Netzwerk Phase 6 – Monitoring, Automatisierung & Troubleshooting

**Dauer:** laufend (ab 2. Ausbildungsjahr) · **Voraussetzung:** Phasen 1–5 (teilweise parallel möglich)

---

## 1. Monitoring

> Tiefgang: [`tools/monitoring-nagios.md`](../tools/monitoring-nagios.md)

- [ ] **Was** überwacht man im Netz?
  - Erreichbarkeit (ICMP), Interface-Status (up/down), Auslastung (bps, %), Fehler/Discards
  - CPU/RAM der Geräte, Temperatur, Netzteile, Lüfter
  - Optik-Werte (Rx/Tx dBm), BGP-/OSPF-Sessions, HSRP-Status
  - Latenz/Jitter/Paketverlust (Smokeping), Zertifikatsablauf
- [ ] **Wie**: SNMP-Polling, SNMP-Traps, Syslog, Streaming Telemetry (gNMI), Flow-Daten
- [ ] Schwellwerte: Warning vs. Critical, Hysterese, Flapping-Erkennung
- [ ] Abhängigkeiten (Parent/Child) – damit bei Ausfall eines Switches nicht 48 Server-Alarme kommen
- [ ] Alarmierung: E-Mail, SMS, PagerDuty/Opsgenie, Rufbereitschaft, Eskalationsstufen
- [ ] Tools im Netzwerkumfeld: **Nagios / Icinga / Checkmk**, **Zabbix**, LibreNMS, Observium, PRTG, SolarWinds, **Prometheus + snmp_exporter + Grafana**
- [ ] Kapazitätsplanung: 95-Perzentil (Abrechnung bei Transit!), Trends

## 2. Dokumentation als „Source of Truth“

- [ ] **NetBox**: DCIM (Standorte, Racks, Geräte, Kabel) + IPAM (Präfixe, IPs, VLANs, VRFs)
- [ ] Prinzip: Erst in NetBox eintragen → dann konfigurieren (bzw. Konfig aus NetBox generieren)
- [ ] Netzwerkdiagramme (draw.io): physisch (Kabel) vs. logisch (VLAN/IP/Routing)

## 3. Konfigurations-Backup & Versionierung

- [ ] **Oxidized** (oder RANCID): holt regelmäßig alle Gerätekonfigs per SSH und legt sie in **Git** ab
- [ ] Vorteil: `git diff` zeigt, *wer wann was* geändert hat → unverzichtbar bei Störungen nach Changes
- [ ] Git-Grundlagen → [`tools/git.md`](../tools/git.md)
- [ ] Vor jedem Change: `show run` sichern; Junos: `commit confirmed 5` (automatischer Rollback!)

## 4. Automatisierung

### 4.1 Warum?
- Hunderte Switches gleich konfigurieren, ohne Tippfehler
- Compliance prüfen (NTP, SNMP, Banner überall gleich?)
- Schneller und reproduzierbar

### 4.2 Ansible für Netzwerke
> Tiefgang: [`tools/ansible.md`](../tools/ansible.md)

- [ ] Agentenlos, SSH/API, YAML-Playbooks
- [ ] Netzwerk-Collections: `cisco.ios`, `cisco.nxos`, `arista.eos`, `junipernetworks.junos`
- [ ] Connection-Typen: `network_cli`, `netconf`, `httpapi`
- [ ] Module: `*_command` (show-Befehle), `*_config` (Konfig-Zeilen), Resource-Module (`*_vlans`, `*_interfaces`, `*_bgp_global`), `*_facts`
- [ ] Beispiel: alle Switches – VLAN anlegen
```yaml
- name: VLAN 120 auf allen Leafs
  hosts: leafs
  gather_facts: false
  tasks:
    - name: VLAN anlegen
      cisco.ios.ios_vlans:
        config:
          - vlan_id: 120
            name: SRV-STAGING
        state: merged
```

### 4.3 Python
- [ ] Grundlagen: Variablen, Listen, Dicts, Schleifen, Funktionen, Dateien, `try/except`, venv, pip
- [ ] **Netmiko** (SSH zu Netzwerkgeräten), **NAPALM** (herstellerneutrale Getter), **Nornir** (Parallelisierung), `ipaddress`-Modul (Subnetting in Python!), `requests` (REST-APIs), `pynetbox`
```python
from netmiko import ConnectHandler
dev = {"device_type": "cisco_ios", "host": "10.0.0.1", "username": "admin", "password": "…"}
with ConnectHandler(**dev) as conn:
    print(conn.send_command("show ip interface brief"))
```
- [ ] Datenformate: **JSON**, **YAML**, XML; Templates mit **Jinja2**
- [ ] APIs: REST, NETCONF (Port 830) / YANG, RESTCONF, gNMI

### 4.4 NetDevOps-Ausblick
- [ ] Konfig als Code in Git → Pull Request → Review → CI-Test (Batfish, Containerlab) → Deployment
- [ ] Grundzüge Terraform-Provider für Netzwerk/Firewall

## 5. Troubleshooting-Methodik

- [ ] **Bottom-up** (L1 → L7), **Top-down** (L7 → L1), **Divide & Conquer** (Mitte, z. B. ping), **Follow the Path** (Hop für Hop), **Compare** (funktionierendes vs. defektes System), **Was hat sich geändert?** (Change-Log, Git-Diff!)
- [ ] Ablauf:
  1. Problem genau beschreiben (Wer? Was? Seit wann? Immer oder sporadisch?)
  2. Informationen sammeln
  3. Hypothese bilden
  4. Testen (eine Änderung zur Zeit!)
  5. Lösung umsetzen, verifizieren
  6. Dokumentieren
- [ ] **Checkliste „Server X nicht erreichbar“**:
  - [ ] L1: Link up? (`show int status`, `ethtool`) Optikwerte? CRC?
  - [ ] L2: richtiges VLAN? MAC auf dem erwarteten Port gelernt? STP forwarding?
  - [ ] L3: Gateway pingbar? ARP-Eintrag vorhanden? Route vorhanden (beide Richtungen!)? Asymmetrisches Routing?
  - [ ] L4: Port offen? (`ss -tlnp` auf Server, `nc -zv` vom Client) Firewall-Regel/Hit-Counter? Lokale Firewall?
  - [ ] L7: Dienst läuft? DNS löst richtig auf? Zertifikat gültig?
  - [ ] Mitschnitt an beiden Enden, wenn alles „okay“ aussieht

## 6. Praxisaufgaben

- [ ] LibreNMS oder Zabbix im Labor: alle Laborgeräte per SNMPv3, Alarm bei Interface down
- [ ] Oxidized + Git-Repo für die Laborgeräte; Änderung machen und per `git log -p` nachvollziehen
- [ ] NetBox im Labor (Docker) mit allen Geräten, IPs, Kabeln befüllen
- [ ] Ansible-Playbook: NTP-, Syslog-, SNMP-Konfig auf alle Laborswitches; zweites Playbook prüft Compliance
- [ ] Python-Skript: aus einer CSV-Datei (Hostname, IP, VLAN) Switchport-Konfigurationen per Jinja2 generieren
- [ ] Im Betrieb: eine echte Störung mit Methodik aus Abschnitt 5 dokumentieren (anonymisiert in „Meine Notizen“)

## Meine Notizen

