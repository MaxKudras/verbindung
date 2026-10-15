# Monitoring – Nagios, Icinga, Checkmk, Zabbix, Prometheus

## Muss ich Nagios können?

**Das Konzept: ja. Das konkrete Tool: das, was dein Betrieb einsetzt.**

- **Nagios Core** ist der Urvater des Open-Source-Monitorings (seit 1999). Reines Nagios wird heute seltener neu eingeführt, ist aber in vielen RZ noch im Betrieb.
- Sein **Plugin-Modell** (kleines Programm prüft etwas → gibt Exit-Code 0/1/2/3 + Text zurück) ist Standard und lebt weiter in **Icinga 2**, **Checkmk**, **Naemon**, **OP5** – und Nagios-Plugins laufen sogar in Zabbix und Sensu.
- Wer Nagios versteht, versteht: Hosts, Services, Checks, Schwellwerte, States, Benachrichtigungen, Abhängigkeiten – das ist auf **jedes** Monitoring übertragbar.
- Daneben stark verbreitet: **Zabbix** (vor allem im Netzwerk- und Server-Bereich) und **Prometheus + Grafana** (Cloud-native, Kubernetes).

**Empfehlung:** Nagios-/Icinga-Prinzip verstehen → Tool deines Betriebs vertiefen → Prometheus/Grafana kennenlernen.

## Grundbegriffe (für alle Tools)

- [ ] **Host** vs. **Service** (Dienst/Check auf einem Host)
- [ ] **Aktive Checks** (Monitoring fragt) vs. **passive Checks** (Host meldet, z. B. SNMP-Trap, Backup-Job-Ergebnis)
- [ ] **Agent-basiert** (NRPE, NSClient++, Icinga-Agent, Checkmk-Agent, Zabbix-Agent, node_exporter) vs. **agentenlos** (ICMP, SNMP, HTTP, SSH)
- [ ] **Schwellwerte**: Warning / Critical
- [ ] **Soft- vs. Hard-State** (erst nach x Fehlversuchen alarmieren → weniger Fehlalarme)
- [ ] **Flapping**-Erkennung
- [ ] **Abhängigkeiten** (Parent/Child): Switch down → nicht 40 Server-Alarme
- [ ] **Downtimes** (Wartungsfenster eintragen!) und **Acknowledgements** („ich kümmere mich“)
- [ ] **Eskalationen**, Kontaktgruppen, Zeitfenster (Rufbereitschaft)
- [ ] **Metriken / Performance-Daten** → Graphen, Trends, Kapazitätsplanung
- [ ] **Blackbox** (von außen: „ist die Webseite erreichbar?“) vs. **Whitebox** (von innen: CPU, Queue-Länge)

## Nagios-Plugin-Prinzip

| Exit-Code | Status |
|---|---|
| 0 | OK |
| 1 | WARNING |
| 2 | CRITICAL |
| 3 | UNKNOWN |

Ausgabe: `STATUS - Text | perfdata`
```bash
/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /
# DISK OK - free space: / 31245 MB (62% inode=91%);| /=18432MB;40000;45000;0;50000

/usr/lib/nagios/plugins/check_http -H www.example.com -S --sni -C 30    # Zertifikat < 30 Tage → Warning
/usr/lib/nagios/plugins/check_ping -H 10.0.0.1 -w 100,20% -c 500,60%
/usr/lib/nagios/plugins/check_tcp -H 10.0.0.5 -p 5432
/usr/lib/nagios/plugins/check_load -w 8,6,4 -c 16,12,8
/usr/lib/nagios/plugins/check_snmp -H 10.0.0.1 -C public -o ifOperStatus.1 -r 1
/usr/lib/nagios/plugins/check_dns -H www.example.com -s 10.0.0.53
/usr/lib/nagios/plugins/check_ntp_time -H 10.0.0.123
```

**Eigenes Plugin in Bash:**
```bash
#!/usr/bin/env bash
# check_backup_age – meldet, wenn die letzte Backup-Datei zu alt ist
FILE=$(ls -t /backup/*.tar.gz 2>/dev/null | head -1)
[ -z "$FILE" ] && { echo "CRITICAL - kein Backup gefunden"; exit 2; }
AGE_H=$(( ( $(date +%s) - $(stat -c %Y "$FILE") ) / 3600 ))
if   [ "$AGE_H" -ge 48 ]; then echo "CRITICAL - Backup ${AGE_H}h alt | age=${AGE_H}h;26;48"; exit 2
elif [ "$AGE_H" -ge 26 ]; then echo "WARNING - Backup ${AGE_H}h alt | age=${AGE_H}h;26;48"; exit 1
else echo "OK - Backup ${AGE_H}h alt | age=${AGE_H}h;26;48"; exit 0; fi
```

**Nagios-Objektdefinition (Beispiel):**
```
define host {
    use         linux-server
    host_name   srv-web01
    address     10.20.0.11
    parents     leaf01
}
define service {
    use                 generic-service
    host_name           srv-web01
    service_description Disk /
    check_command       check_nrpe!check_disk_root
}
```
```bash
nagios -v /etc/nagios/nagios.cfg     # Konfiguration prüfen vor Reload
```

## Icinga 2 (moderner Nachfolger)

- [ ] Eigene DSL (`object Host`, `apply Service … assign where …` – regelbasiert statt jede Kombination einzeln)
- [ ] Master/Satellite/Agent-Architektur, Icinga Web 2 / Icinga DB Web, Icinga Director (GUI-Konfig)
```bash
icinga2 daemon -C                    # Konfig prüfen
icinga2 object list --type Service --name '*disk*'
```

## Checkmk

- [ ] Basiert auf dem Nagios-Kern (bzw. eigenem CMC), **automatische Service-Erkennung** (Discovery), sehr verbreitet in Deutschland
- [ ] Agent liefert alles in einem Abruf; Web-GUI-Konfiguration; Regeln statt Einzelkonfiguration
```bash
cmk -I srv-web01        # Discovery
cmk -O                  # Konfig aktivieren
cmk -nv srv-web01       # Check-Durchlauf testen
```

## Zabbix

- [ ] Server, Datenbank, Frontend, **Proxy** (für entfernte Standorte/Netzsegmente), Agent / Agent 2
- [ ] **Templates** (Linux by Zabbix agent, Cisco IOS by SNMP …), Items, **Triggers**, Actions, Discovery (LLD – findet automatisch alle Interfaces/Platten)
- [ ] Sehr stark für SNMP-Netzwerkmonitoring
```bash
zabbix_get -s 10.20.0.11 -k system.cpu.load[all,avg1]   # Item vom Agent testen
zabbix_agent2 -t vfs.fs.size[/,pfree]
```

## Prometheus + Grafana

- [ ] **Pull-Modell**: Prometheus holt Metriken per HTTP von **Exportern** (`node_exporter` :9100, `snmp_exporter`, `blackbox_exporter`, `windows_exporter`)
- [ ] Zeitreihen-Datenbank, Labels, **PromQL**
- [ ] **Alertmanager** (Gruppierung, Stummschalten, Routing an E-Mail/Teams/PagerDuty)
- [ ] **Grafana** für Dashboards (kann auch Zabbix, Loki, InfluxDB, Elasticsearch anzeigen)
```promql
up == 0                                                          # Ziele, die nicht erreichbar sind
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)   # CPU %
(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
predict_linear(node_filesystem_avail_bytes{mountpoint="/"}[6h], 24*3600) < 0      # Platte in 24h voll?
rate(node_network_receive_bytes_total[5m]) * 8                   # bit/s
```
```yaml
# Alert-Regel
groups:
  - name: server
    rules:
      - alert: DiskFastVoll
        expr: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) > 0.85
        for: 10m
        labels: { severity: warning }
        annotations: { summary: "{{ $labels.instance }}: / über 85 %" }
```

## Weitere Tools kennen

PRTG (Windows-lastige Umgebungen), LibreNMS/Observium (Netzwerk), SolarWinds, Datadog/Dynatrace (kommerziell, APM), Uptime Kuma (einfach), Smokeping (Latenz), Graylog/ELK/Loki (Logs)

## Was sollte im RZ überwacht werden? (Checkliste)

- [ ] Erreichbarkeit aller Geräte (ICMP), OOB-Interfaces
- [ ] Hardware über Redfish/SNMP/OOB: Netzteile, Lüfter, Temperatur, RAID, Platten
- [ ] Server: CPU, RAM, Swap, Disk (% **und** Prognose), Inodes, Load, Dienste, Prozesse, NTP-Offset
- [ ] Netzwerk: Interface-Status, Auslastung, Errors/Discards, Optik-dBm, BGP/OSPF-Nachbarn, HA-Status
- [ ] Anwendungen: HTTP-Status/Antwortzeit, Zertifikatsablauf, DB-Verbindungen, Queues
- [ ] Backup-Erfolg, Snapshot-Alter, Storage-Füllstand
- [ ] RZ-Infrastruktur: USV, Klima, Temperatur/Feuchte, Stromverbrauch pro Rack/PDU

## Übungen

- [ ] Icinga 2 oder Checkmk Raw (kostenlos) im Labor; alle Laborserver + Switches aufnehmen
- [ ] Eigenes Plugin (`check_backup_age` oben) einbinden
- [ ] Abhängigkeit Switch → Server konfigurieren, Switch abschalten, nur **ein** Alarm?
- [ ] Downtime für Wartung setzen
- [ ] Prometheus + node_exporter + Grafana (Dashboard „Node Exporter Full“) + Alertmanager per Docker Compose
- [ ] Zabbix: Switch per SNMPv3 mit Template aufnehmen, Interface-Discovery beobachten
- [ ] Im Betrieb: Welche Alarme kommen am häufigsten? Welche davon sind Fehlalarme? Verbesserungsvorschlag erarbeiten
