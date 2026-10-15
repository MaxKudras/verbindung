# PowerShell

## Muss ich das können?

**Server: Pflicht (Windows). Netzwerk: nützlich** (Tests von Windows-Clients aus, Reports). PowerShell 7 läuft auch auf Linux. VMware wird per **PowerCLI** automatisiert, Microsoft 365/Entra per PowerShell-Modulen.

## Konzept

- [ ] **Objekte statt Text** – Ausgabe hat Eigenschaften, die man filtern/sortieren kann
- [ ] Cmdlets nach **Verb-Noun**: `Get-`, `Set-`, `New-`, `Remove-`, `Start-`, `Stop-`, `Test-`
- [ ] Windows PowerShell 5.1 (eingebaut) vs. **PowerShell 7** (`pwsh`, plattformübergreifend)
- [ ] Execution Policy (`Get-ExecutionPolicy`, `Set-ExecutionPolicy RemoteSigned`)

## Sich selbst helfen

```powershell
Get-Help Get-Service -Examples
Update-Help
Get-Command *firewall*
Get-Command -Module ActiveDirectory
Get-Service | Get-Member              # welche Eigenschaften/Methoden?
Get-Alias ls ; Get-Alias -Definition Get-ChildItem
```

## Pipeline

```powershell
Get-Service | Where-Object Status -eq 'Running' | Sort-Object DisplayName | Select-Object Name, DisplayName
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 5 Name, @{n='RAM_MB';e={[math]::Round($_.WorkingSet/1MB)}}
Get-ChildItem C:\Logs -Recurse -Filter *.log | Where-Object LastWriteTime -lt (Get-Date).AddDays(-30) | Remove-Item -WhatIf
Get-EventLog -LogName System -Newest 20 | Format-Table -AutoSize
Get-Service | Export-Csv C:\temp\dienste.csv -NoTypeInformation -Encoding UTF8
Import-Csv benutzer.csv | ForEach-Object { $_.Name }
Get-Process | ConvertTo-Json | Out-File p.json
Get-ChildItem | Measure-Object -Property Length -Sum
Select-String -Path C:\Logs\*.log -Pattern "error"     # wie grep
```
> **`-WhatIf`** zeigt, was passieren würde – bei allem, was löscht oder ändert, zuerst so testen. `-Confirm` fragt nach.

## Skript-Grundlagen

```powershell
param(
    [Parameter(Mandatory)] [string[]] $ComputerName,
    [int] $Port = 3389
)
$ErrorActionPreference = 'Stop'

foreach ($c in $ComputerName) {
    try {
        $r = Test-NetConnection -ComputerName $c -Port $Port -WarningAction SilentlyContinue
        [PSCustomObject]@{ Host = $c; Port = $Port; Offen = $r.TcpTestSucceeded }
    } catch {
        Write-Warning "$c : $($_.Exception.Message)"
    }
}
# Aufruf: .\Test-Ports.ps1 -ComputerName srv01,srv02 -Port 443 | Format-Table
```
```powershell
if ($x -gt 5) { } elseif ($x -eq 0) { } else { }
# Vergleich: -eq -ne -gt -ge -lt -le -like 'srv*' -match '^srv\d+' -contains -in
switch ($status) { 'Running' { … } default { … } }
function Get-FreeSpace { param($Drive='C') (Get-PSDrive $Drive).Free / 1GB }
$liste = @('a','b'); $hash = @{ Name='srv01'; IP='10.0.0.1' }; $hash.Name
"Server: $($env:COMPUTERNAME)"           # Variablen in Strings
Start-Transcript C:\temp\log.txt ; Stop-Transcript
```

## Remoting

```powershell
Enable-PSRemoting -Force                       # WinRM aktivieren (in Domänen per GPO)
Enter-PSSession -ComputerName srv01            # interaktiv
Invoke-Command -ComputerName srv01,srv02 -ScriptBlock { Get-Service W32Time }
$s = New-PSSession srv01 ; Copy-Item .\tool.exe -Destination C:\temp -ToSession $s
Get-CimInstance -ComputerName srv01 Win32_OperatingSystem | Select Caption, LastBootUpTime
```

## Nützliche Admin-Einzeiler

```powershell
Get-ComputerInfo | Select OsName, OsVersion, CsName
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime
Get-HotFix | Sort InstalledOn -Desc | Select -First 5
Get-NetIPConfiguration ; Get-NetAdapter ; Get-NetRoute
Test-NetConnection srv01 -Port 445
Resolve-DnsName srv01
Get-NetTCPConnection -State Listen
Get-NetFirewallRule -Enabled True | Where Direction -eq Inbound | Select DisplayName
New-NetFirewallRule -DisplayName "Allow 8443" -Direction Inbound -Protocol TCP -LocalPort 8443 -Action Allow
Get-LocalUser ; Get-LocalGroupMember Administrators
Get-ScheduledTask | Where State -eq Ready
Restart-Computer -ComputerName srv01 -Wait -For PowerShell -Timeout 600
Get-WinEvent -FilterHashtable @{LogName='System'; Level=2; StartTime=(Get-Date).AddDays(-1)}
```

## Module, die du kennen solltest

| Modul | Zweck |
|---|---|
| ActiveDirectory | AD-Verwaltung (RSAT) |
| GroupPolicy | GPOs |
| DnsServer / DhcpServer | Windows-DNS/DHCP |
| Hyper-V / FailoverClusters | Virtualisierung & Cluster |
| VMware.PowerCLI | vSphere |
| Microsoft.Graph | Entra ID / M365 |
| Pester | Tests für Skripte |
| PSScriptAnalyzer | Linter |

```powershell
Install-Module VMware.PowerCLI -Scope CurrentUser
Invoke-ScriptAnalyzer .\skript.ps1
```

## Übungen

- [ ] Report: alle Dienste mit Starttyp „Automatisch“, die nicht laufen
- [ ] AD: 50 Benutzer aus CSV anlegen (siehe [server/03](../server/03-windows-server.md)) und inaktive Konten (> 90 Tage) als CSV exportieren
- [ ] Skript, das auf mehreren Servern per `Invoke-Command` Plattenplatz abfragt und als HTML-Report (`ConvertTo-Html`) speichert
- [ ] Geplante Aufgabe, die das Skript täglich ausführt
- [ ] Skripte in Git, mit PSScriptAnalyzer ohne Warnungen
