# Server Phase 3 – Windows Server

**Dauer:** 6–8 Wochen · **Voraussetzung:** Windows-/PowerShell-Grundlagen aus [00-fundament](../00-fundament.md)

---

## 1. Installation & Grundkonfiguration

- [ ] Versionen: Windows Server 2019 / 2022 / 2025; Editionen **Standard** vs. **Datacenter** (unbegrenzte VMs, Storage Spaces Direct, SDN)
- [ ] **Lizenzierung**: pro Kern (mind. 16 Kerne pro Server, 8 pro CPU), 2 VMs pro Standard-Lizenz, **CALs** (User/Device) – IHK-relevant!
- [ ] Server Core vs. Desktop Experience (Core: weniger Angriffsfläche, weniger Updates)
- [ ] Erstkonfiguration: `sconfig` (Core), Hostname, IP, Domain-Join, Updates, Remotedesktop
- [ ] Verwaltung: **Server-Manager**, **Windows Admin Center**, **RSAT**, PowerShell Remoting
```powershell
Rename-Computer -NewName SRV-DC01 -Restart
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.0.11 -PrefixLength 24 -DefaultGateway 10.10.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.0.11,10.10.0.12
Add-Computer -DomainName firma.local -Restart
Get-WindowsFeature | Where Installed
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

## 2. Active Directory Domain Services

### 2.1 Begriffe
- [ ] **Domäne**, **Struktur (Tree)**, **Gesamtstruktur (Forest)**, **Organisationseinheit (OU)**, **Domänencontroller (DC)**
- [ ] Vertrauensstellungen (Trusts): transitiv, Forest-Trust, External
- [ ] **Global Catalog**, **Standorte (Sites)** & Subnetze, Replikation (intra-site / inter-site, KCC)
- [ ] Schema, Funktionsebenen (Domain/Forest Functional Level)
- [ ] AD-Datenbank `NTDS.dit`, SYSVOL (DFSR-repliziert)
- [ ] Objekte: Benutzer, Gruppen, Computer, Kontakte, Service Accounts (**gMSA**)

### 2.2 FSMO-Rollen (alle fünf nennen können!)
| Rolle | Ebene | Aufgabe |
|---|---|---|
| Schema Master | Forest | Schema-Änderungen |
| Domain Naming Master | Forest | Domänen hinzufügen/entfernen |
| RID Master | Domäne | RID-Pools für neue SIDs |
| PDC Emulator | Domäne | Zeitquelle der Domäne, Passwortänderungen, Kontosperren, GPO-Bearbeitung |
| Infrastructure Master | Domäne | Verweise auf Objekte anderer Domänen |

- [ ] FSMO verschieben (Transfer) vs. übernehmen (Seize)

### 2.3 Gruppen
- [ ] Gruppentypen: Sicherheit vs. Verteilung
- [ ] Bereiche: **Domänenlokal**, **Global**, **Universal**
- [ ] **AGDLP** (bzw. AGUDLP): **A**ccounts → **G**lobale Gruppen (Rolle/Abteilung) → **D**omänen**l**okale Gruppen (Ressource+Recht) → **P**ermissions
  - Beispiel: Max → `G-Buchhaltung` → `DL-Share-Finanzen-RW` → Ändern auf `\\fs01\Finanzen`

### 2.4 Authentifizierung
- [ ] **Kerberos**: KDC (auf DC), TGT, Service Ticket, SPN; **max. 5 Minuten Zeitabweichung** → NTP-Hierarchie (PDC-Emulator an externe Quelle)
- [ ] NTLM (Legacy, möglichst einschränken)
- [ ] LDAP / LDAPS (389/636)
- [ ] **DNS ist für AD lebenswichtig**: SRV-Records (`_ldap._tcp.dc._msdcs.<domäne>`), Clients müssen DCs als DNS nutzen
```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.firma.local
Resolve-DnsName _ldap._tcp.dc._msdcs.firma.local -Type SRV
```

### 2.5 Betrieb & Diagnose
```powershell
dcdiag /v /c /e /q              # Gesundheit aller DCs (nur Fehler)
repadmin /replsummary           # Replikationsübersicht
repadmin /showrepl
repadmin /syncall /AdeP
netdom query fsmo
nltest /dsgetdc:firma.local
nltest /sc_verify:firma.local   # sichere Kanal-Verbindung des Clients
Test-ComputerSecureChannel -Repair
klist ; klist purge             # Kerberos-Tickets
w32tm /query /source
setspn -L svc-sql
Get-ADDomainController -Filter * | Select Name, Site, IsGlobalCatalog
Get-ADReplicationFailure -Target firma.local -Scope Domain
```

### 2.6 Benutzerverwaltung per PowerShell
```powershell
Import-Module ActiveDirectory
Get-ADUser -Identity mmuster -Properties *
Get-ADUser -Filter 'Enabled -eq $false' | Select Name
Get-ADUser -Filter * -Properties LastLogonDate | Where LastLogonDate -lt (Get-Date).AddDays(-90)
New-ADUser -Name "Max Muster" -SamAccountName mmuster -UserPrincipalName mmuster@firma.local `
  -Path "OU=Benutzer,OU=Firma,DC=firma,DC=local" -AccountPassword (Read-Host -AsSecureString) -Enabled $true
Set-ADAccountPassword mmuster -Reset -NewPassword (Read-Host -AsSecureString)
Unlock-ADAccount mmuster
Search-ADAccount -LockedOut
Add-ADGroupMember -Identity G-Buchhaltung -Members mmuster
Get-ADGroupMember DL-Share-Finanzen-RW -Recursive
Get-ADPrincipalGroupMembership mmuster
# Massenanlage aus CSV
Import-Csv users.csv | ForEach-Object { New-ADUser -Name $_.Name -SamAccountName $_.Sam -Path $_.OU -Enabled $true -AccountPassword (ConvertTo-SecureString $_.Pw -AsPlainText -Force) }
```

### 2.7 Sicherheit & Wiederherstellung
- [ ] **AD-Papierkorb** aktivieren, gelöschte Objekte wiederherstellen (`Get-ADObject -IncludeDeletedObjects`, `Restore-ADObject`)
- [ ] Systemstatus-Backup des DC, autoritative vs. nicht autoritative Wiederherstellung (DSRM)
- [ ] **Tier-Modell** (Tier 0 = DCs/AD-Admins), separate Admin-Konten, PAW
- [ ] **LAPS** (Windows LAPS) für lokale Admin-Passwörter
- [ ] Protected Users, Fine-Grained Password Policies, krbtgt-Passwort-Rotation
- [ ] Angriffe kennen: Pass-the-Hash, Kerberoasting, Golden Ticket – Tools wie PingCastle/Purple Knight für Audits

## 3. Gruppenrichtlinien (GPO)

- [ ] Computer- vs. Benutzerkonfiguration; Richtlinien vs. Einstellungen (Preferences)
- [ ] Verarbeitungsreihenfolge **LSDOU**: Lokal → Site → Domain → OU (spätere gewinnt)
- [ ] Vererbung blockieren, **Erzwungen** (Enforced), Sicherheitsfilterung, WMI-Filter, Loopback-Verarbeitung
- [ ] Central Store für ADMX-Vorlagen (`\\firma.local\SYSVOL\firma.local\Policies\PolicyDefinitions`)
- [ ] Typische GPOs: Passwortrichtlinie (Default Domain Policy), Laufwerkszuordnung, Drucker, Windows Update, Firewall, Sicherheitsbaselines (Microsoft Security Compliance Toolkit)
```powershell
gpupdate /force
gpresult /r
gpresult /h C:\temp\gp.html
Get-GPO -All | Select DisplayName, GpoStatus
Backup-GPO -All -Path C:\GPO-Backup
Get-GPOReport -Name "Server-Baseline" -ReportType Html -Path C:\temp\r.html
```

## 4. Infrastrukturrollen

### DNS (Windows)
- [ ] AD-integrierte Zonen, Forward-/Reverse-Zonen, Weiterleitungen, bedingte Weiterleitungen, Scavenging (Bereinigung alter Einträge)
```powershell
Get-DnsServerZone
Get-DnsServerResourceRecord -ZoneName firma.local -Name srv-web01
Add-DnsServerResourceRecordA -ZoneName firma.local -Name srv-app01 -IPv4Address 10.10.0.50 -CreatePtr
Clear-DnsClientCache ; ipconfig /flushdns ; ipconfig /registerdns
```
### DHCP (Windows)
- [ ] Bereiche, Reservierungen, Optionen, **DHCP-Failover** (Load Balance / Hot Standby), Autorisierung im AD
```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 10.20.0.0
Add-DhcpServerv4Reservation -ScopeId 10.20.0.0 -IPAddress 10.20.0.50 -ClientId "00-50-56-aa-bb-cc"
```

### Dateiserver
- [ ] **NTFS-Rechte vs. Freigaberechte** – effektiv gilt die restriktivere Kombination (IHK-Klassiker!)
- [ ] Vererbung, explizit vs. geerbt, „Verweigern“ schlägt „Zulassen“
- [ ] Access-Based Enumeration, Schattenkopien (VSS), Kontingente (FSRM), Dateiprüfung
- [ ] **DFS-Namespaces** und **DFS-Replikation**
- [ ] SMB-Versionen: **SMB1 deaktivieren**, SMB3-Verschlüsselung
```powershell
Get-SmbShare ; New-SmbShare -Name Finanzen -Path D:\Shares\Finanzen -FullAccess "FIRMA\DL-Share-Finanzen-RW"
Get-SmbSession ; Get-SmbOpenFile
icacls D:\Shares\Finanzen
Get-Acl D:\Shares\Finanzen | Format-List
Set-SmbServerConfiguration -EnableSMB1Protocol $false
```

### Weitere Rollen
- [ ] **WSUS** (Update-Genehmigung, Gruppen, Berichte) bzw. modernes Patchen mit Intune/Azure Update Manager/Configuration Manager
- [ ] **AD CS** – eigene Zertifizierungsstelle (Root offline, Issuing CA), Vorlagen, Auto-Enrollment, CRL-Verteilung
- [ ] **IIS** (Websites, Bindings, Application Pools, Zertifikate)
- [ ] **RDS** (Remotedesktopdienste: Session Host, Broker, Gateway, Lizenzierung)
- [ ] **Druckserver**
- [ ] **Failover-Clustering** (Quorum, Cluster Shared Volumes) – siehe Phase 4
- [ ] **NPS** (RADIUS)

## 5. Überwachung & Fehlersuche

- [ ] **Ereignisanzeige**: System, Anwendung, Sicherheit, Setup; Ereignis-IDs merken:

| ID | Bedeutung |
|---|---|
| 4624 / 4625 | Anmeldung erfolgreich / fehlgeschlagen |
| 4740 | Konto gesperrt |
| 4720 / 4726 | Benutzer erstellt / gelöscht |
| 4728 / 4732 | Mitglied zu globaler / lokaler Gruppe hinzugefügt |
| 1074 | Herunterfahren/Neustart durch Benutzer/Prozess |
| 6005 / 6006 / 6008 | Eventlog gestartet / gestoppt / **unerwartetes Herunterfahren** |
| 41 (Kernel-Power) | Neustart ohne sauberes Herunterfahren |
| 7036 | Dienst gestartet/gestoppt |

```powershell
Get-WinEvent -LogName System -MaxEvents 50
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625; StartTime=(Get-Date).AddHours(-1)}
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2}       # Kritisch + Fehler
Get-Service | Where Status -eq 'Stopped'
Get-Process | Sort CPU -Descending | Select -First 10
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 2 -MaxSamples 5
Get-HotFix | Sort InstalledOn -Descending | Select -First 10
Get-CimInstance Win32_LogicalDisk | Select DeviceID, @{n='FreeGB';e={[math]::Round($_.FreeSpace/1GB,1)}}
```
- [ ] Leistungsüberwachung (perfmon), Ressourcenmonitor (resmon), Task-Manager
- [ ] **Sysinternals**: Process Explorer, Process Monitor, Autoruns, TCPView, PsExec
- [ ] Systemreparatur: `sfc /scannow`, `DISM /Online /Cleanup-Image /RestoreHealth`, `chkdsk /f`
- [ ] Netzwerk: `Test-NetConnection`, `netstat -ano`, `Get-NetTCPConnection`, `Resolve-DnsName`
- [ ] Remoteverwaltung: `Enter-PSSession srv01`, `Invoke-Command -ComputerName srv01,srv02 -ScriptBlock { … }`, WinRM (5985/5986)

## 6. Hybrid / Cloud (Ausblick)

- [ ] **Microsoft Entra ID** (ehem. Azure AD), **Entra Connect** (Synchronisation), Password Hash Sync / Pass-through, MFA
- [ ] Azure Arc für On-Prem-Server

## 7. Praxisaufgaben

- [ ] Labor-Domäne `labor.local` mit **zwei DCs**, DNS und DHCP-Failover
- [ ] OU-Struktur + 50 Benutzer per CSV und PowerShell anlegen
- [ ] AGDLP-Berechtigungskonzept für 3 Abteilungsfreigaben umsetzen und mit Testbenutzer prüfen
- [ ] GPOs: Laufwerkszuordnung, Passwortrichtlinie, Firewall, Hintergrundbild (zum Testen der Verarbeitung)
- [ ] Einen DC abschalten, Auswirkungen beobachten, FSMO-Rollen verschieben
- [ ] Gelöschten Benutzer über AD-Papierkorb wiederherstellen
- [ ] Zweistufige PKI (Offline-Root + Issuing CA) und Zertifikat für IIS ausstellen
- [ ] Microsoft-Zertifikat optional: AZ-800/AZ-801 (Windows Server Hybrid Administrator)

## Meine Notizen

