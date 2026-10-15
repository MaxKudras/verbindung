# Lernpfade Rechenzentrum – Fachinformatiker Systemintegration

Persönliche Lernnotizen für die Ausbildung im Rechenzentrum, mit zwei Schwerpunkten:

- **Netzwerkabteilung** → [`netzwerk/`](netzwerk/README.md)
- **Serverabteilung** → [`server/`](server/README.md)

Jede Datei ist als **Checkliste** aufgebaut (`- [ ]` → `- [x]` abhaken, sobald du es *erklären und vorführen* kannst – nicht nur gelesen hast).

## Aufbau

| Ordner / Datei | Inhalt |
|---|---|
| [`00-fundament.md`](00-fundament.md) | Was du für **beide** Bereiche brauchst: Zahlensysteme, OSI, Linux-/Windows-Basis, RZ-Infrastruktur, ITIL |
| [`netzwerk/`](netzwerk/README.md) | 6 Phasen: Grundlagen → Switching → Routing → Dienste → Sicherheit → Monitoring & Automatisierung |
| [`server/`](server/README.md) | 6 Phasen: Hardware/OOB → Linux → Windows Server → Virtualisierung → Storage & Backup → Container & IaC |
| [`tools/`](tools/README.md) | Tiefgang zu einzelnen Werkzeugen: `netstat`/`ss`, `dig`, `tcpdump`/Wireshark, SSH, Git, Ansible, Nagios & Monitoring, PowerShell, Bash-Scripting |
| [`labor/`](labor/README.md) | Heimlabor aufbauen + 12 Praxisprojekte mit Abnahmekriterien |
| [`ihk/`](ihk/README.md) | Prüfungsrelevante Themen (AP1 / AP2), Rechenaufgaben, Abschlussprojekt-Ideen |

## Häufige Frage: „Muss ich X wirklich können?“

Kurze Antworten auf die Frage *„Muss ich das im RZ wirklich können?“* – Details in [`tools/`](tools/README.md):

| Tool | Muss ich das können? | Warum |
|---|---|---|
| `netstat` | **Ja, lesen können** – auf Linux aber `ss` bevorzugen | Windows nutzt weiterhin `netstat -ano`; auf Linux ist `netstat` (Paket *net-tools*) veraltet, `ss` ist der Nachfolger. Ältere Server und Anleitungen nutzen es noch. |
| `dig` | **Ja, Pflicht** | DNS ist an *jeder zweiten* Störung beteiligt. `dig` ist das Standard-Diagnosewerkzeug für DNS. |
| Git | **Ja, Pflicht** | Konfigurationen, Skripte, Ansible-Playbooks, Doku – alles wird heute versioniert. Ohne Git kein IaC. |
| Ansible | **Ja, sehr empfehlenswert** | Der De-facto-Standard für Automatisierung von Servern **und** Netzwerkgeräten. Hebt dich stark von anderen Azubis ab. |
| Nagios | **Konzept ja, Tool je nach Betrieb** | Nagios selbst ist alt, aber sein Plugin-Modell lebt in Icinga, Checkmk, Naemon weiter. Wenn du Nagios verstehst, verstehst du fast jedes Monitoring. |
| `tcpdump` / Wireshark | **Ja** | Wenn alle anderen Tools „sieht gut aus“ sagen, zeigt der Mitschnitt die Wahrheit. |
| PowerShell | **Ja** (Server), **nützlich** (Netzwerk) | Windows-Administration ohne PowerShell skaliert nicht. |
| Python | **Später, ja** | Für Netzwerkautomatisierung (Netmiko, NAPALM) und komplexere Skripte. Nach Bash. |

## Empfohlene Reihenfolge

```
00-fundament
   ├── netzwerk/01 ──► netzwerk/02 ──► netzwerk/03 ──► netzwerk/04 ──► netzwerk/05 ──► netzwerk/06
   └── server/01  ──► server/02  ──► server/03  ──► server/04  ──► server/05  ──► server/06
                         ▲
   tools/* begleitend: ss/netstat + dig ab Woche 1, Git ab Woche 2, Ansible ab Netzwerk-/Server-Phase 3
```

## Grober Zeitplan über die Ausbildung

| Zeitraum | Netzwerk | Server | Tools | Zertifikat (optional) |
|---|---|---|---|---|
| 1. Jahr, H1 | Fundament, Phase 1 | Fundament, Phase 1 | ss/netstat, dig, ping/traceroute, SSH, Bash-Basics | – |
| 1. Jahr, H2 | Phase 2 | Phase 2 (Linux) | Git, tcpdump, PowerShell-Basics | CompTIA Network+/Linux+ (optional) |
| 2. Jahr, H1 | Phase 3 | Phase 3 (Windows) | Ansible-Basics | **CCNA** |
| 2. Jahr, H2 | Phase 4 | Phase 4 (Virtualisierung) | Monitoring (Nagios/Icinga/Zabbix) | **LPIC-1** oder **RHCSA** |
| 3. Jahr | Phasen 5–6 | Phasen 5–6 | Ansible fortgeschritten, Python | RHCE / CCNP / VCP (nach Ausbildung) |

## Arbeitsweise

1. **Lesen** → Thema in der Datei durchgehen.
2. **Nachbauen** → im Labor ([`labor/`](labor/README.md)) praktisch umsetzen.
3. **Erklären** → einem Kollegen/Ausbilder in eigenen Worten erklären (Feynman-Methode).
4. **Abhaken** → erst dann `[x]`.
5. **Eigene Notizen** → unter jeder Datei gibt es einen Abschnitt *„Meine Notizen“* für Erkenntnisse aus dem Betrieb (bitte keine Passwörter, Kunden-IPs oder internen Hostnamen ins Repo!).
