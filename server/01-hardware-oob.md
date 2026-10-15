# Server Phase 1 – Hardware & Out-of-Band-Management

**Dauer:** 3–4 Wochen · **Voraussetzung:** [00-fundament](../00-fundament.md)

---

## 1. Bauformen

- [ ] Rack-Server (1U, 2U, 4U), Blade-Server (Chassis mit gemeinsamer Strom-/Netz-Infrastruktur), Tower, Multi-Node (z. B. 4 Nodes in 2U), Hyperconverged-Appliances
- [ ] Vor- und Nachteile bezüglich Dichte, Kühlung, Wartung

## 2. Komponenten

### CPU
- [ ] Sockel, Kerne, Threads (Hyper-Threading/SMT), Takt, Cache, TDP
- [ ] Intel Xeon vs. AMD EPYC
- [ ] **NUMA**: jeder Sockel hat „seinen“ RAM – wichtig für VM-Sizing und Datenbanken
- [ ] Virtualisierungsfunktionen: VT-x/AMD-V, VT-d/IOMMU (müssen im BIOS an sein!)

### Arbeitsspeicher
- [ ] **ECC** (Fehlerkorrektur) – Pflicht im Server, RDIMM vs. LRDIMM
- [ ] Speicherkanäle, korrekte Bestückung (Population Rules des Herstellers!), gleiche Module pro Kanal
- [ ] Korrigierbare vs. nicht korrigierbare Fehler (CE/UE) im Eventlog → Modul tauschen

### Netzwerkkarten
- [ ] Onboard (LOM), OCP-Mezzanine, PCIe-Karten; 1/10/25/100G
- [ ] SR-IOV, RDMA/RoCE (Grundzüge), SmartNIC/DPU (Ausblick)

### Storage im Server
- [ ] HDD (SATA, **SAS**, 7.2k/10k/15k U/min), SSD (SATA/SAS), **NVMe** (PCIe; U.2, U.3, M.2, E1.S)
- [ ] DWPD/TBW (Schreibfestigkeit von SSDs), Read-Intensive vs. Mixed-Use
- [ ] Hot-Swap-Einschübe, Backplane
- [ ] Bootmedium: BOSS-Karte (Dell), M.2-RAID, SD-Karten (veraltet für ESXi!)

### Stromversorgung & Kühlung
- [ ] Redundante Netzteile (1+1), Hot-Swap, Effizienzklassen (80 PLUS Platinum/Titanium)
- [ ] Redundante Lüfter, Luftstromrichtung (front-to-back!)

## 3. RAID

- [ ] Hardware-RAID (Controller mit Cache + BBU/Flash-Backup) vs. Software-RAID (mdadm, ZFS, Storage Spaces) vs. HBA/JBOD-Modus (für Ceph/vSAN/ZFS)
- [ ] Cache-Modi: Write-Back (schnell, braucht Akku/Flash-Schutz) vs. Write-Through

| RAID | Min. Platten | Nutzkapazität | Ausfalltoleranz | Bemerkung |
|---|---|---|---|---|
| 0 | 2 | 100 % | 0 | Striping, nur Performance |
| 1 | 2 | 50 % | 1 | Mirroring, z. B. Bootplatten |
| 5 | 3 | (n−1)/n | 1 | Parität; lange Rebuilds bei großen Platten = Risiko |
| 6 | 4 | (n−2)/n | 2 | doppelte Parität |
| 10 | 4 | 50 % | 1 pro Spiegel | Datenbanken, VMs – schnell |
| 50 / 60 | 6 / 8 | | | Stripe über RAID-5/6-Gruppen |

- [ ] **Hot Spare**, Rebuild, Patrol Read, Consistency Check
- [ ] Rechenaufgabe: 8 × 4 TB in RAID 6 → nutzbar? (24 TB) In RAID 10? (16 TB)
- [ ] **RAID ist kein Backup!** (schützt nicht vor Löschen, Ransomware, Controllerdefekt, Brand)
- [ ] Tools: Dell `perccli`, Broadcom `storcli64 /c0 show`, HPE `ssacli ctrl all show config`, Linux `cat /proc/mdstat`

## 4. Firmware & Boot

- [ ] **BIOS vs. UEFI**, GPT vs. MBR, **Secure Boot**, TPM 2.0
- [ ] Boot-Reihenfolge, PXE-Boot (Phase 2/4)
- [ ] BIOS-Einstellungen: Virtualisierung, Power-Profile (Performance vs. Energiesparen), Sub-NUMA-Clustering
- [ ] Firmware-Updates: BIOS, BMC, RAID-Controller, NICs, Platten – Hersteller-Tools (Dell Lifecycle Controller / OpenManage, HPE SPP, Lenovo XClarity), Reihenfolge und Kompatibilitätsmatrix beachten

## 5. Out-of-Band-Management (OOB / BMC)

> Mit dem BMC kommst du an den Server, auch wenn das Betriebssystem hängt oder das Produktionsnetz weg ist. Im RZ täglich genutzt.

- [ ] Dell **iDRAC**, HPE **iLO**, Lenovo **XClarity Controller**, Supermicro **IPMI/BMC**, Cisco CIMC
- [ ] Funktionen: Remote-Konsole (KVM), virtuelle Medien (ISO mounten), Power on/off/reset, Hardware-Sensoren, System Event Log (SEL), Firmware-Update, Alarmierung
- [ ] Eigenes OOB-Netz (nie ins Internet!), BMC-Firmware aktuell halten (bekannte Sicherheitslücken!)
- [ ] Standards: **IPMI** (alt, unsicher in v1.5), **Redfish** (REST-API, modern)

```bash
# IPMI
ipmitool -I lanplus -H 10.99.1.21 -U admin -P '***' chassis status
ipmitool -I lanplus -H 10.99.1.21 -U admin -P '***' power status|on|off|cycle|soft
ipmitool -I lanplus -H 10.99.1.21 -U admin -P '***' sel list
ipmitool -I lanplus -H 10.99.1.21 -U admin -P '***' sensor
ipmitool -I lanplus -H 10.99.1.21 -U admin -P '***' sol activate   # serielle Konsole
ipmitool lan print 1                                                # lokal auf dem Server

# Dell racadm
racadm -r 10.99.1.21 -u root -p '***' getsysinfo
racadm getsel
racadm serveraction powercycle
racadm storage get pdisks

# Redfish (curl)
curl -sk -u admin:'***' https://10.99.1.21/redfish/v1/Systems/ | jq
```

### Hardware-Infos vom laufenden Linux aus
```bash
dmidecode -t system           # Hersteller, Modell, Seriennummer
dmidecode -t memory           # RAM-Module
lscpu
lsblk -o NAME,SIZE,TYPE,MODEL,SERIAL
lspci | grep -i -E 'ether|raid|nvme'
lshw -short
nvme list ; nvme smart-log /dev/nvme0
smartctl -a /dev/sda
sensors                       # lm-sensors
```
### Windows
```powershell
Get-CimInstance Win32_BIOS            # Seriennummer
Get-CimInstance Win32_ComputerSystem
Get-PhysicalDisk
Get-CimInstance Win32_PhysicalMemory | Select BankLabel, Capacity, Speed
```

## 6. RZ-Handwerk (praktisch!)

- [ ] Server auspacken, Schienen montieren, einbauen (zu zweit bei schweren Servern!), Kabelarm
- [ ] **ESD-Schutz** (Armband)
- [ ] Verkabeln: 2× Strom (A+B), Netzwerk, OOB, Beschriftung beider Kabelenden
- [ ] Komponententausch im laufenden Betrieb: Platte, Netzteil, Lüfter – **vorher Status-LEDs und Controller prüfen, richtige Platte identifizieren (Locate-LED blinken lassen!)**
- [ ] Nicht hot-swap-fähig (RAM, CPU, Mainboard): Wartungsfenster, Server herunterfahren, VMs evakuieren
- [ ] **RMA**-Prozess: Fehler dokumentieren (SEL-Log, TSR/AHS-Report), Case beim Hersteller, Teil tauschen, Altteil zurück (Datenschutz: Platten ggf. behalten – „Keep your Hard Drive“)
- [ ] Asset-Management: Seriennummer, Service-Tag, Garantie/Support-Ende in CMDB/NetBox
- [ ] Datenträgervernichtung nach DIN 66399
- [ ] Rückbau (Decommissioning): Daten löschen, aus Monitoring/Backup/DNS/CMDB entfernen

## 7. Praxisaufgaben

- [ ] Mit Ausbilder einen Server einbauen und in Betrieb nehmen (inkl. Doku)
- [ ] OOB-Konsole öffnen, ISO mounten, BIOS-Einstellungen durchgehen
- [ ] SEL eines Servers mit `ipmitool` oder racadm auslesen und Einträge erklären
- [ ] RAID-Level-Entscheidung begründen: Datenbankserver, Fileserver, Backup-Repository
- [ ] Plattentausch in einem RAID begleiten und den Rebuild verfolgen

## Meine Notizen

