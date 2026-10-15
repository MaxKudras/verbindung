# tcpdump & Wireshark – Paketmitschnitte

## Muss ich das können?

**Ja (Netzwerk: Pflicht, Server: sehr wichtig).** Wenn Firewall-Team, Server-Team und Anwendung alle sagen „bei uns ist alles ok“, beweist der Mitschnitt, was wirklich auf der Leitung passiert. Im RZ läuft das typisch so: `tcpdump` auf dem Server/Firewall mitschneiden → `.pcap`-Datei → in Wireshark am Arbeitsplatz analysieren.

> ⚠️ Mitschnitte können personenbezogene Daten und Passwörter enthalten. Nur mit Auftrag/Freigabe, gezielt filtern, Dateien nach Analyse löschen (DSGVO!).

## tcpdump

```bash
tcpdump -D                                   # verfügbare Interfaces
tcpdump -i eth0                              # alles auf eth0 (Vorsicht auf produktiven Servern: viel!)
tcpdump -i eth0 -nn                          # keine Namens-/Portauflösung (immer nutzen)
tcpdump -i eth0 -nn host 10.0.0.5
tcpdump -i eth0 -nn src 10.0.0.5 and dst port 443
tcpdump -i eth0 -nn net 10.20.0.0/16
tcpdump -i eth0 -nn port 53                  # DNS
tcpdump -i eth0 -nn 'tcp[tcpflags] & (tcp-syn|tcp-rst) != 0'   # nur SYN/RST
tcpdump -i eth0 -nn icmp
tcpdump -i eth0 -nn arp
tcpdump -i eth0 -nn vlan                     # getaggte Frames
tcpdump -i eth0 -nn -e                       # MAC-Adressen anzeigen
tcpdump -i eth0 -nn -A port 80               # Inhalt als ASCII
tcpdump -i any -nn port 5060                 # alle Interfaces
tcpdump -i eth0 -nn -c 100 host 10.0.0.5     # nach 100 Paketen stoppen
tcpdump -i eth0 -nn -s 128 -w /tmp/cap.pcap host 10.0.0.5     # nur Header-Anfang speichern
tcpdump -i eth0 -w /tmp/cap-%H%M.pcap -G 300 -W 12 port 443   # Ringpuffer: 12 Dateien à 5 min
tcpdump -nn -r /tmp/cap.pcap                 # Datei lesen
tcpdump -nn -r /tmp/cap.pcap 'tcp port 443' | head
```
Filter-Logik (BPF): `and` / `or` / `not`, Klammern in Anführungszeichen: `'host 10.0.0.5 and (port 80 or port 443)'`

### Ausgabe lesen
```
10:15:01.123 IP 10.0.0.10.51514 > 10.0.0.5.443: Flags [S], seq 123, win 64240, length 0
10:15:01.124 IP 10.0.0.5.443 > 10.0.0.10.51514: Flags [S.], seq 456, ack 124, length 0
10:15:01.124 IP 10.0.0.10.51514 > 10.0.0.5.443: Flags [.], ack 457, length 0
```
Flags: `[S]` SYN, `[S.]` SYN-ACK, `[.]` ACK, `[P.]` PSH-ACK (Daten), `[F.]` FIN, `[R]` RST

**Diagnosemuster:**
| Beobachtung | Bedeutung |
|---|---|
| Nur `[S]`, keine Antwort, Wiederholungen | Paket wird unterwegs verworfen (Firewall/Route) oder Ziel antwortet nicht |
| `[S]` → `[R.]` | Ziel erreichbar, aber **Port geschlossen** (oder Firewall sendet Reset) |
| SYN kommt am Server an, SYN-ACK geht raus, Client sieht es nie | **Rückweg** kaputt / asymmetrisches Routing |
| Handshake ok, große Pakete hängen | **MTU-Problem** |
| Viele Retransmissions | Paketverlust, Überlast, Duplex |

## Wireshark

- [ ] Oberfläche: Paketliste, Paketdetails (Schichten aufklappbar), Hex-Ansicht
- [ ] **Capture-Filter** (BPF wie tcpdump, beim Aufzeichnen) vs. **Display-Filter** (eigene Syntax, nachträglich)

### Display-Filter
```
ip.addr == 10.0.0.5
ip.src == 10.0.0.5 && tcp.dstport == 443
tcp.port == 22
!(arp || dns)
dns
dns.flags.rcode != 0                 # DNS-Fehler (NXDOMAIN, SERVFAIL)
http.request
http.response.code >= 400
tls.handshake.type == 1              # Client Hello
tls.handshake.extensions_server_name contains "example"
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.reset == 1
tcp.analysis.retransmission
tcp.analysis.flags                   # alle TCP-Auffälligkeiten
icmp.type == 3
eth.addr == 00:50:56:aa:bb:cc
vlan.id == 110
frame.time_delta > 1                 # Pausen > 1 s
```

### Nützliche Funktionen
- [ ] **Follow TCP Stream** (Rechtsklick → Folgen → TCP-Stream)
- [ ] Statistik → **Conversations**, **Endpoints**, **Protocol Hierarchy**, **I/O Graph**
- [ ] Analyse → **Expert Information**
- [ ] Zeitanzeige umstellen (seit vorherigem Paket)
- [ ] Spalten hinzufügen (z. B. `tcp.stream`)
- [ ] TLS entschlüsseln mit SSLKEYLOGFILE (nur im Labor)

### tshark (Wireshark auf der CLI)
```bash
tshark -r cap.pcap -Y 'http.request' -T fields -e ip.src -e http.host -e http.request.uri
tshark -i eth0 -f 'port 53' -Y 'dns.flags.rcode != 0'
```

## Mitschnitt auf Netzwerkgeräten

- [ ] **SPAN** (Port Mirroring), RSPAN, **ERSPAN** (über GRE zu einem Analyse-Server)
- [ ] Netzwerk-TAPs (passiv)
- [ ] Gerätemitschnitte: Cisco EPC (`monitor capture`), FortiGate `diagnose sniffer packet`, Palo Alto Packet Capture, Juniper `monitor traffic interface`
- [ ] VMware: `pktcap-uw`; Kubernetes: `kubectl debug` + tcpdump

## Übungen

- [ ] Eigenen HTTP- und HTTPS-Aufruf mitschneiden: DNS, TCP-Handshake, TLS Client/Server Hello, HTTP – jede Phase markieren
- [ ] Port-Scan im Labor mitschneiden (`nmap -sS`) und SYN/RST-Muster erkennen
- [ ] DHCP-DORA mitschneiden
- [ ] Gleichzeitig auf Client **und** Server mitschneiden und ein Paket „verschwinden“ lassen (Firewallregel) – nachweisen, wo es verloren geht
- [ ] Wireshark-Übungsdateien: wiki.wireshark.org/SampleCaptures, malware-traffic-analysis.net (fortgeschritten)
