# netstat & ss – Verbindungen und offene Ports

## Muss ich das können?

**Ja.** Die Frage *„Lauscht der Dienst überhaupt?“* und *„Wer ist gerade mit dem Server verbunden?“* stellst du dir im RZ ständig.

- **Linux:** `netstat` gehört zum Paket **net-tools**, das seit Jahren als veraltet gilt und auf neuen Systemen oft gar nicht mehr installiert ist. Der Nachfolger ist **`ss`** (Paket *iproute2*, immer vorhanden). → **`ss` aktiv nutzen, `netstat` lesen können** (alte Server, Anleitungen, Kollegen).
- **Windows:** `netstat` ist dort weiterhin **das** Standardwerkzeug, daneben PowerShell `Get-NetTCPConnection`.
- **Netzwerkgeräte:** Entsprechungen sind z. B. `show tcp brief`, `show sockets`, `show control-plane host open-ports` (Cisco).

## Die wichtigste Zeile

```bash
ss -tulpn
```
| Option | Bedeutung |
|---|---|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | nur lauschende (LISTEN) Sockets |
| `-p` | Prozess anzeigen (braucht root/sudo) |
| `-n` | numerisch (keine DNS-/Portnamen-Auflösung → schneller, eindeutig) |

Netstat-Äquivalent: `netstat -tulpn` (gleiche Buchstaben!).

Beispielausgabe lesen:
```
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
tcp   LISTEN 0      511    0.0.0.0:80          0.0.0.0:*         users:(("nginx",pid=812,fd=6))
tcp   LISTEN 0      128    127.0.0.1:5432      0.0.0.0:*         users:(("postgres",pid=640,fd=5))
tcp   LISTEN 0      128    [::]:22             [::]:*            users:(("sshd",pid=590,fd=4))
```
- `0.0.0.0:80` → Nginx lauscht auf **allen** IPv4-Adressen
- `127.0.0.1:5432` → PostgreSQL **nur lokal** erreichbar (häufige Ursache für „Datenbank von außen nicht erreichbar“ – und oft so gewollt!)
- `[::]:22` → SSH auf allen IPv6- (und meist auch IPv4-) Adressen

## ss – weitere Befehle

```bash
ss -tan                              # alle TCP-Verbindungen numerisch
ss -tan state established            # nur aufgebaute Verbindungen
ss -tan state time-wait | wc -l      # wie viele TIME_WAIT?
ss -tn dst 10.0.0.5                  # Verbindungen zu einem Ziel
ss -tn '( dport = :443 or sport = :443 )'
ss -tnp | grep nginx
ss -s                                # Zusammenfassung / Statistik
ss -ti                               # TCP-Interna (RTT, cwnd, Retransmits)
ss -xl                               # Unix-Sockets
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c   # Verbindungen pro Zustand
ss -tn state established | awk 'NR>1 {split($5,a,":"); print a[1]}' | sort | uniq -c | sort -rn | head   # Top-Clients
```

## netstat – Linux (Legacy) und Windows

```bash
# Linux (falls net-tools installiert)
netstat -tulpn          # lauschende Ports + Prozess
netstat -tan            # alle TCP-Verbindungen
netstat -rn             # Routing-Tabelle (heute: ip route)
netstat -i              # Interface-Statistik (heute: ip -s link)
netstat -s              # Protokollstatistiken (heute: nstat)
```
```powershell
# Windows
netstat -ano                      # alle Verbindungen + PID
netstat -ano | findstr :443
netstat -anob                     # + Programmname (Admin-Rechte)
netstat -r                        # Routing-Tabelle
netstat -e -s                     # Statistiken
tasklist /FI "PID eq 1234"        # PID zum Prozess auflösen
```
```powershell
# PowerShell – objektbasiert, besser filterbar
Get-NetTCPConnection -State Listen | Sort LocalPort
Get-NetTCPConnection -LocalPort 443
Get-NetTCPConnection -State Established | Group RemoteAddress | Sort Count -Desc | Select -First 10
Get-NetUDPEndpoint
Get-Process -Id (Get-NetTCPConnection -LocalPort 3389).OwningProcess
```

## Vergleichstabelle

| Aufgabe | Linux neu | Linux alt | Windows |
|---|---|---|---|
| Lauschende Ports + Prozess | `ss -tulpn` | `netstat -tulpn` | `netstat -ano` / `Get-NetTCPConnection -State Listen` |
| Aufgebaute Verbindungen | `ss -tn state established` | `netstat -tn` | `netstat -an \| findstr ESTABLISHED` |
| Routing-Tabelle | `ip route` | `netstat -rn` / `route -n` | `route print` |
| Interface-Statistik | `ip -s link` | `netstat -i` / `ifconfig` | `netstat -e` / `Get-NetAdapterStatistics` |
| ARP-Tabelle | `ip neigh` | `arp -n` | `arp -a` |
| Welcher Prozess nutzt Port? | `ss -lptn 'sport = :80'` / `lsof -i :80` / `fuser 80/tcp` | `netstat -tulpn \| grep :80` | `netstat -ano \| findstr :80` |

## TCP-Zustände verstehen

| Zustand | Bedeutung | Auffällig, wenn … |
|---|---|---|
| LISTEN | wartet auf Verbindungen | fehlt → Dienst läuft nicht / falscher Port / falsche IP |
| SYN_SENT | Client hat SYN geschickt, keine Antwort | viele → Ziel/Firewall verwirft Pakete |
| SYN_RECV | Server hat SYN/ACK geschickt | sehr viele → SYN-Flood |
| ESTABLISHED | aktive Verbindung | |
| FIN_WAIT_1/2 | aktiv schließende Seite wartet | |
| CLOSE_WAIT | Gegenseite hat geschlossen, **eigene Anwendung schließt nicht** | viele → Bug/Leak in der Anwendung |
| TIME_WAIT | Verbindung beendet, wartet 2×MSL | sehr viele (zehntausende) → Port-Erschöpfung möglich |
| LAST_ACK | | |

## lsof – „Alles ist eine Datei“

```bash
lsof -i :443              # wer nutzt Port 443
lsof -i TCP -s TCP:LISTEN
lsof -i @10.0.0.5         # Verbindungen zu Host
lsof -p 812               # alle offenen Dateien eines Prozesses
lsof +L1                  # gelöschte, noch offene Dateien (Platte voll, obwohl du gelöscht hast!)
```

## Typische Szenarien

1. **„Webseite nicht erreichbar“** → `ss -tlnp | grep -E ':80|:443'` – lauscht Nginx? Auf welcher IP?
2. **„Port belegt, Dienst startet nicht“** (`Address already in use`) → `ss -lptn 'sport = :8080'` → welcher Prozess blockiert?
3. **„Server langsam, viele Verbindungen“** → `ss -s`, Verbindungen pro Zustand und pro Client zählen
4. **„Ist die DB-Verbindung vom App-Server aufgebaut?“** → `ss -tn dst <db-ip>:5432`
5. **Sicherheits-Check** → `ss -tulpn` – lauscht etwas, das nicht soll (z. B. Redis auf `0.0.0.0`)?

## Übungen

- [ ] Auf einer Labor-VM Nginx installieren; mit `ss -tulpn` prüfen; dann `listen 127.0.0.1:80;` setzen und Unterschied sehen
- [ ] Mit `python3 -m http.server 8000` einen Port öffnen und ihn mit `ss`, `lsof` und `fuser` finden
- [ ] Unter Windows: Welcher Prozess lauscht auf 135, 445, 3389? (`netstat -ano` + `tasklist`)
- [ ] Mit `curl` in einer Schleife viele Verbindungen erzeugen und TIME_WAIT zählen
