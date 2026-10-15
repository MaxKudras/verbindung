# IHK-Prüfung – Fachinformatiker Systemintegration

Stand der Prüfungsordnung seit 2020: **Abschlussprüfung Teil 1 (AP1)** etwa zur Mitte der Ausbildung und **Teil 2 (AP2)** am Ende. Aktuelle Details immer bei der eigenen IHK bzw. im Prüfungskatalog der AkA prüfen.

## Aufbau

| Teil | Inhalt | Gewichtung |
|---|---|---|
| AP1 | „Einrichten eines IT-gestützten Arbeitsplatzes“ (schriftlich, 90 min) | 20 % |
| AP2 – Betriebliche Projektarbeit | Projekt (40 h), Dokumentation, Präsentation + Fachgespräch | 50 % |
| AP2 – Konzeption und Administration von IT-Systemen | schriftlich, 90 min | 10 % |
| AP2 – Analyse und Entwicklung von Netzwerken | schriftlich, 90 min | 10 % |
| AP2 – Wirtschafts- und Sozialkunde | schriftlich, 60 min | 10 % |

## Themen, die immer wieder kommen (mit Verweis auf die Lernpfade)

### Netzwerk
- [ ] Subnetting IPv4 + IPv6 (Präfixe, Adressen kürzen) → [netzwerk/01](../netzwerk/01-grundlagen-adressierung.md)
- [ ] OSI-Modell, Protokolle und Ports zuordnen → [00-fundament](../00-fundament.md), [netzwerk/01](../netzwerk/01-grundlagen-adressierung.md)
- [ ] VLAN, Trunk, STP, LACP → [netzwerk/02](../netzwerk/02-switching.md)
- [ ] Routing-Tabelle lesen/ergänzen, statische Routen, NAT → [netzwerk/03](../netzwerk/03-routing.md)
- [ ] DNS, DHCP (DORA) → [netzwerk/04](../netzwerk/04-netzwerkdienste.md)
- [ ] Firewall-Regeltabellen ausfüllen, DMZ, VPN/IPsec → [netzwerk/05](../netzwerk/05-sicherheit.md)
- [ ] WLAN (802.11-Standards, WPA2/WPA3, 802.1X) – im RZ weniger, in der Prüfung schon!
- [ ] Verkabelung, Übertragungsraten, Glasfasertypen → [00-fundament](../00-fundament.md)

### Server & Systeme
- [ ] **RAID-Level** + Kapazitätsberechnung → [server/01](../server/01-hardware-oob.md)
- [ ] **Backup-Arten** (Voll/Inkr./Diff.), Generationenprinzip, 3-2-1 → [server/05](../server/05-storage-backup.md)
- [ ] **USV-Typen** (VFD/VI/VFI) + Überbrückungszeit berechnen → [00-fundament](../00-fundament.md)
- [ ] Virtualisierung (Hypervisor-Typen, Vorteile), Container vs. VM → [server/04](../server/04-virtualisierung.md), [server/06](../server/06-container-iac.md)
- [ ] Active Directory, Gruppenkonzept AGDLP, NTFS- vs. Freigaberechte → [server/03](../server/03-windows-server.md)
- [ ] Linux-Rechte (chmod oktal), Befehle → [00-fundament](../00-fundament.md)
- [ ] SAN/NAS/DAS, iSCSI, Cloud-Modelle (IaaS/PaaS/SaaS, Public/Private/Hybrid)
- [ ] Monitoring, SNMP → [tools/monitoring-nagios.md](../tools/monitoring-nagios.md)
- [ ] Pseudocode / einfache Skripte lesen & schreiben → [tools/bash-scripting.md](../tools/bash-scripting.md), [tools/powershell.md](../tools/powershell.md)

### IT-Sicherheit & Datenschutz
- [ ] Schutzziele (CIA), **Schutzbedarfsfeststellung** (normal/hoch/sehr hoch), BSI IT-Grundschutz
- [ ] DSGVO-Grundsätze, **TOM** (technisch-organisatorische Maßnahmen), Auftragsverarbeitung
- [ ] Verschlüsselung (symmetrisch/asymmetrisch/hybrid), Hash, Signatur, PKI, Zertifikate
- [ ] Angriffsarten (Phishing, Ransomware, DDoS, Social Engineering, MITM) und Gegenmaßnahmen
- [ ] Authentifizierung: MFA, Passwortrichtlinien

### Wirtschaft & Projekt
- [ ] **Angebotsvergleich / Nutzwertanalyse**
- [ ] **Amortisationsrechnung**, Kostenvergleich (Kauf vs. Leasing vs. Miete/Cloud), Abschreibung (linear)
- [ ] Stromkosten berechnen (W × h × €/kWh) – z. B. Server 24/7
- [ ] Projektmanagement: Netzplan, Gantt, kritischer Pfad, Wasserfall vs. Scrum, Lasten-/Pflichtenheft
- [ ] ITIL / SLA / Verfügbarkeit berechnen
- [ ] Kundenkommunikation, Präsentation

## Rechenaufgaben zum Üben

1. **Stromkosten:** 12 Server à 350 W laufen 24/7. Strompreis 0,28 €/kWh. Jahreskosten?
   <details><summary>Lösung</summary>12 × 0,35 kW × 8760 h = 36.792 kWh × 0,28 € = **10.301,76 €**</details>
2. **USV:** Last 2,4 kW, USV-Akku liefert 4,8 kWh nutzbar. Überbrückungszeit (vereinfachend)?
   <details><summary>Lösung</summary>4,8 / 2,4 = **2 h**</details>
3. **RAID:** 6 Platten à 8 TB in RAID 5, RAID 6, RAID 10 – nutzbare Kapazität?
   <details><summary>Lösung</summary>RAID 5: 40 TB, RAID 6: 32 TB, RAID 10: 24 TB</details>
4. **Übertragungszeit:** 500 GB Backup über 1 Gbit/s (80 % nutzbar) – wie lange?
   <details><summary>Lösung</summary>500 GB × 8 = 4000 Gbit / 0,8 Gbit/s = 5000 s ≈ **1 h 23 min**</details>
5. **Verfügbarkeit:** Zwei Komponenten seriell mit je 99,5 %; danach dieselben parallel.
   <details><summary>Lösung</summary>Seriell: 0,995² = 99,0025 %; parallel: 1 − 0,005² = 99,9975 %</details>
6. **Amortisation:** Neue Storage kostet 24.000 €, spart 800 €/Monat an Strom und Wartung. Wann amortisiert?
   <details><summary>Lösung</summary>24.000 / 800 = **30 Monate**</details>
7. **Subnetting:** 10.50.0.0/22 in 8 gleich große Subnetze – Präfix, Hosts je Netz, drittes Subnetz?
   <details><summary>Lösung</summary>/25, 126 Hosts, drittes Netz 10.50.1.0/25</details>

## Ideen für das Abschlussprojekt (RZ-typisch)

Gute Projekte haben ein **echtes Problem**, **Alternativen mit Entscheidung (Nutzwertanalyse)**, **Wirtschaftlichkeitsbetrachtung**, **Umsetzung** und **Qualitätssicherung/Test**.

- Einführung eines zentralen Monitorings (z. B. Migration Nagios → Checkmk/Icinga) für einen Server-/Netzbereich
- Automatisierte Konfigurationssicherung von Netzwerkgeräten (Oxidized + Git) inkl. Änderungsbenachrichtigung
- Aufbau eines Ansible-basierten Server-Provisionings (Template + Baseline-Rolle)
- Migration eines Hypervisor-Clusters (z. B. VMware → Proxmox) für eine Testumgebung
- Einführung eines Backup-Konzepts mit Immutable Repository gegen Ransomware
- Aufbau eines neuen Kunden-Racks inkl. Netzwerkanbindung (VLAN/VRF, Firewall, Monitoring, Doku in NetBox)
- Einführung von NetBox als DCIM/IPAM inkl. Datenübernahme
- Ablösung von SNMPv2c durch SNMPv3 im gesamten Netzbereich
- Zentrales Log-Management (Graylog/OpenSearch) mit Alarmierung

**Tipp:** Projektidee früh mit Ausbilder absprechen, Projektantrag sauber formulieren, Zeitplan (40 h) realistisch, Doku während des Projekts schreiben.

## Lernressourcen für die Prüfung

- Alte Prüfungen (über Ausbildungsbetrieb/IHK/U-Form-Verlag)
- Buch-Reihen speziell für FISI-Prüfungen (z. B. Westermann, Europa-Lehrmittel)
- YouTube: „Fachinformatiker Prüfungsvorbereitung“-Kanäle
- Lerngruppe mit anderen Azubis
