# Server Phase 5 – Storage & Backup

**Dauer:** 4–6 Wochen · **Voraussetzung:** Phasen 1, 2, 4

---

## 1. Storage-Architekturen

- [ ] **DAS** (Direct Attached), **NAS** (File über Netzwerk: NFS/SMB), **SAN** (Block über dediziertes Netz: FC/iSCSI/NVMe-oF)
- [ ] **Block** vs. **File** vs. **Object** Storage – Einsatzzwecke
- [ ] Storage-Arrays: Controller (Dual, Active/Active vs. Active/Passive), Disk Shelves, Cache
- [ ] Hersteller: NetApp (ONTAP), Dell (PowerStore, PowerMax, Unity), HPE (Alletra, 3PAR/Primera), Pure Storage, IBM, Hitachi
- [ ] Software-defined Storage: **Ceph**, vSAN, Storage Spaces Direct, ZFS (TrueNAS)

## 2. Kennzahlen

- [ ] **IOPS**, **Latenz** (ms/µs), **Durchsatz** (MB/s), Blockgröße, Lese-/Schreibverhältnis, Random vs. Sequential
- [ ] Queue Depth
- [ ] Faustwerte: HDD 7.2k ≈ 80 IOPS, 15k ≈ 180 IOPS, SATA-SSD ≈ zehntausende, NVMe ≈ hunderttausende
- [ ] Messen: `fio`, `iostat -x`, esxtop (DAVG/KAVG/GAVG), Windows `diskspd`
```bash
fio --name=randrw --rw=randrw --bs=4k --size=2G --iodepth=32 --numjobs=4 --runtime=60 --group_reporting --direct=1
```

## 3. Funktionen

- [ ] **LUN**, LUN-Masking, Host-Gruppen/Initiator-Gruppen
- [ ] **Thin** vs. Thick Provisioning (Überbuchung überwachen!), UNMAP/TRIM
- [ ] **Deduplizierung**, **Kompression**, Data Reduction Ratio
- [ ] **Tiering** (SSD/HDD), Caching
- [ ] **Snapshots** (Array-basiert, Copy-on-Write vs. Redirect-on-Write)
- [ ] **Replikation**: synchron (RPO 0, Distanzlimit/Latenz) vs. asynchron; Metro-Cluster/Stretched Cluster
- [ ] Erasure Coding (Ceph, Object Storage)

## 4. SAN-Protokolle

### Fibre Channel
- [ ] **WWN / WWPN / WWNN**, HBA, FC-Switches (Brocade, Cisco MDS)
- [ ] **Zoning** (Single-Initiator-Zoning), Fabric A und B getrennt (Redundanz!)
- [ ] Geschwindigkeiten 16/32/64G
- [ ] Brocade-Basics: `switchshow`, `zoneshow`, `nsshow`, `porterrshow`

### iSCSI
- [ ] Initiator, Target, **IQN**, Portal (IP:3260), CHAP, eigenes VLAN/Netz, Jumbo Frames
```bash
iscsiadm -m discovery -t sendtargets -p 10.40.0.10
iscsiadm -m node --login
iscsiadm -m session -P 3
cat /etc/iscsi/initiatorname.iscsi
```
```powershell
Get-IscsiTarget ; Get-IscsiSession ; iscsicpl
```

### Multipathing
- [ ] Warum: mehrere Pfade (2 HBAs × 2 Controller) → Redundanz + Performance
- [ ] ALUA (optimierte/nicht optimierte Pfade), Policies: Round Robin, Fixed, MRU
```bash
multipath -ll
multipathd show paths
```
```powershell
Get-MSDSMSupportedHW ; mpclaim -s -d
```

### NVMe-oF, FCoE (Grundzüge)

## 5. File-Protokolle

- [ ] **NFS**: v3 (zustandslos, UDP/TCP, Rechte über UID/GID) vs. **v4/4.1** (zustandsbehaftet, nur TCP 2049, Kerberos möglich); Export-Optionen `rw`, `sync`, `root_squash`, `no_root_squash` (Risiko!)
```bash
showmount -e nfs01
mount -t nfs4 nfs01:/export/data /mnt/data
nfsstat -m
```
- [ ] **SMB**: SMB 2/3 (SMB1 aus!), Multichannel, Verschlüsselung, Continuous Availability

## 6. Object Storage

- [ ] **S3-API** (Buckets, Objekte, Keys, Versionierung, Lifecycle-Policies, **Object Lock** für unveränderliche Backups)
- [ ] MinIO, Ceph RGW, NetApp StorageGRID, Cloud (AWS S3, Azure Blob)
```bash
aws s3 ls s3://backup-bucket --endpoint-url https://s3.firma.local
mc ls minio/backup         # MinIO-Client
```

## 7. Ceph (Grundzüge, im modernen RZ häufig)

- [ ] Komponenten: MON, MGR, **OSD** (pro Platte), MDS (für CephFS), RGW (S3)
- [ ] RADOS, Pools, Placement Groups, **CRUSH-Map**, Replikation (size 3 / min_size 2) vs. Erasure Coding
- [ ] Zugriff: RBD (Block), CephFS (File), RGW (Object)
```bash
ceph -s ; ceph health detail
ceph osd tree ; ceph osd df
ceph df
```

## 8. Backup

### 8.1 Strategie
- [ ] **3-2-1-1-0-Regel**: 3 Kopien, 2 verschiedene Medien, 1 extern (anderer Brandabschnitt/Standort), 1 offline/unveränderlich (air-gapped/immutable), 0 Fehler bei Wiederherstellungstests
- [ ] **RPO** (Recovery Point Objective – maximal tolerierter Datenverlust) und **RTO** (Recovery Time Objective – maximale Ausfallzeit)
- [ ] Backup-Arten (**IHK-Klassiker!**):

| Art | Was wird gesichert? | Backup-Dauer | Restore benötigt | Archivbit |
|---|---|---|---|---|
| Voll | alles | lang | nur das Vollbackup | zurückgesetzt |
| Inkrementell | Änderungen seit **letzter Sicherung (egal welcher)** | kurz | Voll + **alle** Inkremente | zurückgesetzt |
| Differenziell | Änderungen seit **letzter Vollsicherung** | wächst täglich | Voll + **letztes** Differenzielles | nicht zurückgesetzt |

- [ ] Synthetic Full, Forever Incremental, Changed Block Tracking (CBT)
- [ ] Rotationsschemata: **Generationenprinzip (Großvater-Vater-Sohn)**, Türme von Hanoi
- [ ] Aufbewahrung (Retention), gesetzliche Fristen (GoBD, HGB: 6/10 Jahre), DSGVO-Löschpflichten vs. Backups
- [ ] Anwendungskonsistenz: VSS (Windows), Pre-/Post-Skripte, DB-Dumps oder Agent (SQL, Exchange, Oracle RMAN)
- [ ] Backup-Verschlüsselung, Schlüsselverwaltung (Schlüssel nicht nur im gesicherten System aufbewahren!)

### 8.2 Medien & Ziele
- [ ] Disk (Deduplizierungs-Appliances wie Data Domain, StoreOnce), **Band (LTO-8/9/10, LTFS, WORM)**, Object Storage/Cloud
- [ ] **Hardened Linux Repository** (Veeam), Object Lock, Air-Gap gegen **Ransomware**
- [ ] Backup-Netz getrennt, Backup-Server **nicht** in der Produktivdomäne (Angreifer löschen zuerst Backups!)

### 8.3 Software
- [ ] **Veeam Backup & Replication** (sehr verbreitet: Jobs, Repositories, Proxies, SureBackup, Instant Recovery)
- [ ] Commvault, Veritas NetBackup, IBM Storage Protect (TSM), Cohesity, Rubrik
- [ ] Open Source: **Bareos**/Bacula, **Borg**, **Restic**, Proxmox Backup Server, `rsync`
```bash
# Restic-Beispiel
restic -r /mnt/backup/repo init
restic -r /mnt/backup/repo backup /etc /var/www
restic -r /mnt/backup/repo snapshots
restic -r /mnt/backup/repo restore latest --target /tmp/restore
restic -r /mnt/backup/repo forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune

# Datenbanken
mysqldump --single-transaction --all-databases | gzip > /backup/db-$(date +%F).sql.gz
pg_dumpall | gzip > /backup/pg-$(date +%F).sql.gz
```

### 8.4 Restore – der eigentliche Zweck
- [ ] Wiederherstellungstests planen und **dokumentieren** (Datei, VM, Datenbank, kompletter Server, AD)
- [ ] Instant Recovery (VM direkt aus dem Backup starten)
- [ ] Bare-Metal-Recovery
- [ ] Runbook: „Wiederherstellung nach Ransomware“

## 9. Disaster Recovery & Business Continuity

- [ ] BCM (Business Continuity Management, BSI 200-4), Notfallhandbuch
- [ ] DR-Standort: Cold / Warm / Hot Site
- [ ] Georedundanz (Mindestabstand nach BSI-Empfehlung: 200 km für georedundante RZ)
- [ ] DR-Tests (Failover-Übungen), Replikation von VMs (Veeam Replication, Zerto, SRM)

## 10. Praxisaufgaben

- [ ] TrueNAS oder Linux-NFS + iSCSI-Target (targetcli) im Labor; Proxmox/ESXi damit verbinden, Multipath mit 2 Netzen
- [ ] Ceph-Cluster in Proxmox mit 3 Knoten
- [ ] Backup-Konzept für das eigene Labor schreiben (RPO/RTO je System, 3-2-1-1-0) und umsetzen (Proxmox Backup Server oder Veeam Community Edition + Restic für Configs)
- [ ] **Restore-Test**: VM löschen und aus Backup wiederherstellen; einzelne Datei; MariaDB-Datenbank
- [ ] IHK-Übung: Wochenplan mit Voll (So) + Inkrementell (Mo–Sa) – welche Bänder brauchst du für einen Restore am Donnerstag? Dasselbe mit Differenziell.

## Meine Notizen

