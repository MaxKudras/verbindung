# 00 – Fundament (für Netzwerk UND Server)

**Dauer:** ca. 4–6 Wochen · **Ziel:** Die Basis, auf der beide Lernpfade aufbauen.

---

## 1. Zahlensysteme

- [ ] Binär ↔ Dezimal ↔ Hexadezimal im Kopf umrechnen (0–255 sicher)
- [ ] Die Zweierpotenzen bis 2^16 auswendig: 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024 … 65536
- [ ] Ein Oktett = 8 Bit, ein Hex-Zeichen = 4 Bit (Nibble)
- [ ] Bit vs. Byte, SI (kB = 1000) vs. IEC (KiB = 1024) – wichtig bei Speicher- und Bandbreitenrechnungen
- [ ] Bandbreite: 1 Gbit/s ≈ 125 MB/s (theoretisch)

**Übung:** `11000000.10101000.00001010.00000001` → `192.168.10.1`; `0xFF` → 255; `0x1F` → 31.

| Dezimal | Binär | Hex |
|---|---|---|
| 128 | 1000 0000 | 80 |
| 192 | 1100 0000 | C0 |
| 224 | 1110 0000 | E0 |
| 240 | 1111 0000 | F0 |
| 248 | 1111 1000 | F8 |
| 252 | 1111 1100 | FC |
| 254 | 1111 1110 | FE |
| 255 | 1111 1111 | FF |

---

## 2. Referenzmodelle

- [ ] **OSI-Modell** (7 Schichten) – Merksatz: *„Alle Deutschen Studenten Trinken Verschiedene Sorten Bier“* (Anwendung, Darstellung, Sitzung, Transport, Vermittlung, Sicherung, Bitübertragung – von oben nach unten)
- [ ] **TCP/IP-Modell** (4 Schichten: Netzzugang, Internet, Transport, Anwendung)
- [ ] Zu jeder Schicht: PDU-Name, Beispielprotokolle, Geräte, Adressen

| OSI | Name | PDU | Protokolle | Geräte | Adresse |
|---|---|---|---|---|---|
| 7 | Anwendung | Daten | HTTP, DNS, SSH, SMTP | – | – |
| 6 | Darstellung | Daten | TLS (teilweise), ASCII, JPEG | – | – |
| 5 | Sitzung | Daten | RPC, NetBIOS | – | – |
| 4 | Transport | Segment / Datagramm | TCP, UDP | Firewall (L4) | Port |
| 3 | Vermittlung | Paket | IPv4, IPv6, ICMP, OSPF | Router, L3-Switch | IP-Adresse |
| 2 | Sicherung | Frame | Ethernet, 802.1Q, ARP*, STP | Switch, Bridge | MAC-Adresse |
| 1 | Bitübertragung | Bit | 1000BASE-T, 10GBASE-SR | Kabel, Hub, Transceiver | – |

\* ARP liegt zwischen L2 und L3.

- [ ] **Encapsulation** verstehen: Daten → TCP-Header → IP-Header → Ethernet-Header + Trailer (FCS)
- [ ] **Troubleshooting nach Schichten**: L1 (Link-LED, Kabel, Optik) → L2 (VLAN, MAC) → L3 (IP, Route) → L4 (Port, Firewall) → L7 (Dienst, DNS)

---

## 3. Linux-Grundlagen (Pflicht für beide Bereiche)

> Netzwerker arbeiten auf Jump-Hosts, Monitoring-Servern, Netzwerk-OS (Arista EOS, Cumulus, SONiC sind Linux!). Linux ist für **beide** Pfade Pflicht.

### 3.1 Dateisystem-Hierarchie (FHS)
- [ ] `/etc` Konfiguration · `/var/log` Logs · `/var/lib` Daten von Diensten · `/home` Benutzer · `/root` · `/opt` Drittsoftware · `/tmp` · `/usr/bin` Programme · `/proc` und `/sys` Kernel-Infos · `/dev` Geräte · `/boot` · `/mnt` und `/media`

### 3.2 Befehle – Navigation & Dateien
```bash
pwd                       # wo bin ich
ls -lah                   # alles, lesbare Größen
cd -                      # zurück zum vorherigen Verzeichnis
cp -a quelle ziel         # kopieren inkl. Rechte/Zeitstempel
mv alt neu
rm -r verzeichnis         # VORSICHT, kein Papierkorb
mkdir -p a/b/c
touch datei
ln -s ziel link           # symbolischer Link
find /var/log -name "*.log" -mtime -1      # Logs, die heute geändert wurden
find / -type f -size +1G 2>/dev/null       # große Dateien finden
file datei                # Dateityp erkennen
stat datei                # Metadaten
```

### 3.3 Befehle – Text verarbeiten (täglich gebraucht!)
```bash
cat / less / head -n 20 / tail -n 50
tail -f /var/log/syslog                   # live mitlesen
grep -i "error" /var/log/syslog
grep -rn "Listen" /etc/nginx/             # rekursiv mit Zeilennummer
grep -v "^#" datei | grep -v "^$"         # Kommentare & Leerzeilen entfernen
wc -l datei
sort | uniq -c | sort -rn                 # Häufigkeiten zählen (Klassiker!)
cut -d: -f1 /etc/passwd                   # Feld 1, Trenner ":"
awk '{print $1}' access.log               # erste Spalte
awk -F: '$3 >= 1000 {print $1}' /etc/passwd
sed 's/alt/neu/g' datei                   # ersetzen (Ausgabe)
sed -i.bak 's/alt/neu/g' datei            # in Datei ersetzen mit Backup
diff -u a.conf b.conf
```
**Übung:** Die 10 IP-Adressen mit den meisten Zugriffen aus einem Webserver-Log:
`awk '{print $1}' access.log | sort | uniq -c | sort -rn | head`

### 3.4 Rechte & Benutzer
- [ ] `rwx` für User/Group/Other, oktal (r=4, w=2, x=1): `chmod 640`, `chmod u+x`
- [ ] Sonderrechte: SUID (4), SGID (2), Sticky Bit (1, z. B. `/tmp`)
- [ ] `chown user:gruppe datei`, `chgrp`, `umask`
- [ ] Benutzer: `useradd -m -s /bin/bash max`, `usermod -aG sudo max`, `passwd`, `userdel -r`
- [ ] `/etc/passwd`, `/etc/shadow`, `/etc/group` – Aufbau erklären können
- [ ] `sudo`, `su -`, `visudo`, `id`, `whoami`, `who`, `w`, `last`, `lastlog`
- [ ] ACLs: `getfacl`, `setfacl -m u:max:rw datei`

### 3.5 Shell-Mechanik
- [ ] Pipes `|`, Umleitungen `>`, `>>`, `2>`, `2>&1`, `&>`, `/dev/null`
- [ ] `tee` (Ausgabe anzeigen UND speichern)
- [ ] Umgebungsvariablen: `echo $PATH`, `export VAR=wert`, `env`
- [ ] History: `history`, `Ctrl+R` (Rückwärtssuche), `!!`, `sudo !!`
- [ ] Tab-Completion, `Ctrl+C`, `Ctrl+Z`, `bg`, `fg`, `jobs`, `nohup`
- [ ] `man befehl`, `befehl --help`, `tldr befehl`
- [ ] Terminal-Multiplexer: `tmux` (Sitzungen überleben SSH-Abbrüche – im RZ Gold wert!)

### 3.6 Editor
- [ ] `vim`: `i` (Einfügen), `Esc`, `:w`, `:q`, `:wq`, `:q!`, `/suche`, `n`, `dd`, `yy`, `p`, `u`, `:set number`, `:%s/alt/neu/g`
- [ ] `nano` als Notlösung

---

## 4. Windows-Grundlagen

- [ ] CMD-Basics: `dir`, `cd`, `copy`, `xcopy`/`robocopy`, `del`, `type`, `findstr`, `tasklist`, `taskkill /PID x /F`, `systeminfo`, `hostname`, `whoami /all`
- [ ] PowerShell-Konzept: **Objekte statt Text**, Verb-Noun (`Get-Process`), Pipeline
- [ ] `Get-Help Befehl -Examples`, `Get-Command *dns*`, `Get-Member`, `Select-Object`, `Where-Object`, `Sort-Object`, `Format-Table`
- [ ] MMC-Snap-ins: `services.msc`, `eventvwr.msc`, `compmgmt.msc`, `diskmgmt.msc`, `devmgmt.msc`, `lusrmgr.msc`
- [ ] Siehe [`tools/powershell.md`](tools/powershell.md)

---

## 5. Rechenzentrums-Infrastruktur

### 5.1 Rack & Raum
- [ ] 19"-Rack, Höheneinheit (1 HE / 1 U = 44,45 mm), typische Racks 42–48 HE
- [ ] Rack-Tiefe, Lastgrenzen (kg), Schienen (Rails), Kabelführung (Kabelmanagement-Arme)
- [ ] **Kaltgang / Warmgang** (Cold/Hot Aisle), Einhausung (Containment), Blindplatten – *warum* fehlende Blindplatten die Kühlung ruinieren
- [ ] Doppelboden, Kabeltrassen über dem Rack
- [ ] Rack-Elevation-Diagramm lesen (welcher Server in welcher HE)

### 5.2 Stromversorgung
- [ ] **A/B-Einspeisung**: zwei unabhängige Stromkreise, jedes Gerät mit 2 Netzteilen an beiden
- [ ] **USV-Typen** (IHK-relevant!): Offline/Standby (VFD), Line-Interactive (VI), Online/Doppelwandler (VFI) – Unterschiede, Umschaltzeit
- [ ] **NEA** (Netzersatzanlage / Dieselgenerator), Startzeit, wie lange die USV überbrücken muss
- [ ] PDU (Basic, Metered, Switched), Steckertypen C13/C14, C19/C20, CEE
- [ ] Leistung: W = V × A, VA vs. W (Leistungsfaktor), kW pro Rack
- [ ] Rechnung: Server zieht 400 W, Rack hat 16 A bei 230 V = 3680 W → wie viele Server? (Reserve für Ausfall eines Strangs beachten!)

### 5.3 Klima & Effizienz
- [ ] CRAC (Kältemittel) / CRAH (Kaltwasser), freie Kühlung
- [ ] Empfohlene Zulufttemperatur (ASHRAE: 18–27 °C)
- [ ] **PUE** = Gesamtenergie RZ / Energie IT-Geräte (ideal → 1,0; gut < 1,4)

### 5.4 Verfügbarkeit & Normen
- [ ] **Tier I–IV** (Uptime Institute) und **EN 50600** (Verfügbarkeitsklassen 1–4)
- [ ] Redundanz-Begriffe: N, N+1, 2N, 2N+1
- [ ] Verfügbarkeit rechnen:

| Verfügbarkeit | Ausfall pro Jahr | Ausfall pro Monat |
|---|---|---|
| 99 % | 3,65 Tage | 7,3 h |
| 99,9 % | 8,76 h | 43,8 min |
| 99,99 % | 52,6 min | 4,4 min |
| 99,999 % | 5,26 min | 26 s |

- [ ] Serielle vs. parallele Verfügbarkeit: seriell A × B; parallel 1 − (1−A)(1−B)
- [ ] MTBF, MTTR; Verfügbarkeit = MTBF / (MTBF + MTTR)

### 5.5 Verkabelung
- [ ] Kupfer: Cat5e (1G), Cat6 (1G/10G bis 55 m), **Cat6A** (10G bis 100 m), Cat7, Cat8 (25/40G bis 30 m)
- [ ] Glasfaser: **Singlemode** (OS2, gelb, lange Strecken, 1310/1550 nm) vs. **Multimode** (OM3 türkis, OM4 türkis/violett, OM5 limettengrün, 850 nm)
- [ ] Stecker: **LC** (Standard im RZ), SC, **MPO/MTP** (12/24 Fasern, für 40/100/400G)
- [ ] Transceiver: SFP (1G), SFP+ (10G), SFP28 (25G), QSFP+ (40G), QSFP28 (100G), QSFP-DD/OSFP (400G/800G)
- [ ] Optik-Kürzel: SR (short reach, MM), LR (long reach, SM, 10 km), ER (40 km), ZR (80 km)
- [ ] DAC (Direct Attach Copper, kurz, günstig), AOC (Active Optical Cable)
- [ ] Polarität bei Glasfaser (Tx → Rx), Reinigung von Steckern (Click-Cleaner!) – **Schmutz ist Fehlerursache Nr. 1 bei Optiken**
- [ ] Strukturierte Verkabelung, Patchfelder, Beschriftungskonzept, Kabelfarben-Konvention im eigenen RZ erfragen

### 5.6 Physische Sicherheit & Brandschutz
- [ ] Zutrittskontrolle (Vereinzelungsanlage, Karte + PIN/Biometrie), Besucherprotokoll
- [ ] Brandfrüherkennung **VESDA** (Ansaugrauchmelder), Löschgas (Inergen, Argon, Novec/FM-200), Sauerstoffreduktion
- [ ] Verhalten bei Alarm (Gaslöschung = Raum sofort verlassen!)

---

## 6. IT-Service-Management & Arbeitsweise

- [ ] **ITIL 4 Grundbegriffe**: Incident (Störung), Problem (Ursache), Change (Änderung), Service Request, Known Error, CMDB
- [ ] Incident-Priorität = Auswirkung × Dringlichkeit
- [ ] **SLA / OLA / UC**, Reaktionszeit vs. Lösungszeit
- [ ] Change-Arten: Standard, Normal, Emergency; CAB; Wartungsfenster; **Rollback-Plan immer vorher!**
- [ ] Vier-Augen-Prinzip bei kritischen Changes
- [ ] Dokumentation: Netzwerkpläne (draw.io), Rack-Elevations, Runbooks/Betriebshandbücher, Wiki
- [ ] Root-Cause-Analysis: 5-Why, Ishikawa, Post-Mortem ohne Schuldzuweisung (blameless)
- [ ] Datenschutz & Informationssicherheit: DSGVO-Grundlagen, TOM, ISO 27001, BSI IT-Grundschutz (Schutzziele **Vertraulichkeit, Integrität, Verfügbarkeit**)

---

## Abschlusscheck Fundament

- [ ] Ich kann ein Paket auf dem Weg vom Browser zum Webserver durch alle OSI-Schichten erklären.
- [ ] Ich finde auf einem Linux-Server per Kommandozeile heraus, welche Dateien in `/var` am meisten Platz belegen (`du -xh /var | sort -h | tail`).
- [ ] Ich kann erklären, warum jeder Server zwei Netzteile auf zwei Strängen hat und was bei Ausfall von Strang A passiert.
- [ ] Ich kann den Unterschied zwischen Incident und Problem an einem Beispiel aus meinem Betrieb erklären.

## Meine Notizen

<!-- Erkenntnisse aus dem Betrieb hier eintragen -->
