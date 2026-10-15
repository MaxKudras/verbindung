# ping, traceroute, mtr, nc, curl – Erreichbarkeit prüfen

## Muss ich das können?

**Ja – das sind die ersten Werkzeuge bei jeder „geht nicht“-Meldung.** Wichtig ist aber, die Grenzen zu kennen: *ping geht nicht* heißt nicht *Host ist down* (ICMP kann gefiltert sein), und *ping geht* heißt nicht *Dienst funktioniert*.

## ping

```bash
ping -c 4 10.0.0.1             # 4 Pakete
ping -i 0.2 -c 50 10.0.0.1     # schneller, für Verlustmessung
ping -I eth1 10.0.0.1          # über bestimmtes Interface / Quell-IP
ping -M do -s 1472 10.0.0.1    # MTU 1500 testen (DF-Bit, 1472+28)
ping -M do -s 8972 10.0.0.1    # Jumbo 9000 testen
ping -6 2001:db8::1 ; ping6
ping -D 10.0.0.1               # mit Zeitstempel (zum Loggen von Aussetzern)
```
```powershell
ping -n 4 10.0.0.1
ping -t 10.0.0.1               # Dauerping
ping -f -l 1472 10.0.0.1       # DF + Größe
Test-Connection 10.0.0.1 -Count 4
```
**Ausgabe interpretieren:** `time=` (RTT), `ttl=` (verrät grob das OS: 64 Linux, 128 Windows, 255 Netzwerkgeräte), Paketverlust %, `Destination Host Unreachable` (ARP schlägt fehl – Host im eigenen Netz nicht da), `Request timed out` (keine Antwort – Filter oder Host weg), `Frag needed` (MTU-Problem).

## traceroute / tracert / mtr

- [ ] Funktionsweise: Pakete mit TTL 1, 2, 3 … → jeder Router antwortet mit ICMP *Time Exceeded*
- [ ] Linux: UDP (Standard), `-I` ICMP, `-T` TCP (besser durch Firewalls)
- [ ] Windows `tracert` nutzt ICMP
- [ ] `* * *` heißt nur: dieser Hop antwortet nicht – **nicht unbedingt** ein Problem, solange spätere Hops antworten
```bash
traceroute -n 8.8.8.8
traceroute -T -p 443 -n www.example.com
tracepath 10.0.0.1             # zeigt auch PMTU
mtr -n 8.8.8.8                 # traceroute + ping live
mtr -rwc 100 -n 8.8.8.8        # Report mit 100 Paketen – ideal für Tickets an Provider
```
```powershell
tracert -d 8.8.8.8
pathping -n 8.8.8.8
```
**mtr lesen:** Verlust nur auf einem Zwischenhop, aber nicht danach → Router priorisiert ICMP niedrig (harmlos). Verlust ab einem Hop **bis zum Ziel** → echtes Problem ab dort.

## Port-Tests (Schicht 4)

```bash
nc -zv 10.0.0.5 443                 # TCP-Port offen?
nc -zvu 10.0.0.5 53                 # UDP (unzuverlässig, UDP antwortet nicht immer)
nc -zv 10.0.0.5 20-25               # Bereich
nc -l 9000                          # Gegenstelle zum Testen öffnen (auf Server B)
timeout 3 bash -c '</dev/tcp/10.0.0.5/443' && echo offen   # ohne nc
telnet 10.0.0.5 25                  # alter Trick, wenn nichts anderes da ist
```
```powershell
Test-NetConnection 10.0.0.5 -Port 443
Test-NetConnection www.example.com -TraceRoute
```

## Anwendungsebene (Schicht 7)

```bash
curl -I https://www.example.com                     # nur Header
curl -v https://www.example.com                     # TLS-Handshake + Header
curl -k https://10.0.0.5                            # Zertifikat ignorieren (nur Test!)
curl --resolve www.example.com:443:10.0.0.5 https://www.example.com   # an bestimmte IP, korrekter Hostname (SNI)
curl -o /dev/null -s -w '%{http_code} %{time_connect} %{time_total}\n' https://www.example.com
curl -x http://proxy:3128 https://example.com       # über Proxy
wget -O- http://10.0.0.5/health
openssl s_client -connect www.example.com:443 -servername www.example.com </dev/null | openssl x509 -noout -dates
```
```powershell
Invoke-WebRequest https://www.example.com -UseBasicParsing | Select StatusCode
```

## Weitere

```bash
arping -I eth0 10.0.0.20       # L2-Erreichbarkeit / doppelte IP finden (-D)
iperf3 -s                      # Bandbreitentest: Server
iperf3 -c 10.0.0.5 -P 4 -t 30  # Client, 4 Streams
nmap -Pn -p 22,443 10.0.0.5    # mehrere Ports (mit Freigabe!)
```

## Entscheidungsbaum

```
ping Ziel-IP ok? ──nein──► ping Gateway ok? ──nein──► lokales Problem (IP, VLAN, Kabel, ARP)
   │                            └─ja──► traceroute/mtr: wo bricht es ab? Route? Firewall?
   ja
   ▼
nc -zv Ziel Port ok? ──nein──► Dienst lauscht? (ss -tlnp auf Ziel) / Firewall (Host + Netz)
   │
   ja
   ▼
curl -v / Anwendung ok? ──nein──► DNS (dig), TLS (Zertifikat), App-Logs, Proxy
```

## Übungen

- [ ] MTU-Test zwischen zwei Labor-VMs, dann MTU auf einer Seite auf 1400 setzen und Effekt beobachten
- [ ] `mtr`-Report zu einer entfernten Seite erstellen und jede Spalte erklären
- [ ] iperf3 zwischen zwei VMs, mit 1 und mit 8 Streams vergleichen
- [ ] Mit `curl -w` Antwortzeiten eines Webservers messen und `time_connect` vs. `time_starttransfer` erklären
