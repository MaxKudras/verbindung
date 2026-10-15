# Werkzeuge – Tiefgang

Jede Datei beantwortet: **Muss ich das können? Wofür im RZ? Welche Befehle? Wie übe ich?**

| Datei | Werkzeug(e) | Relevanz Netzwerk | Relevanz Server |
|---|---|---|---|
| [netstat-ss.md](netstat-ss.md) | `netstat`, `ss`, `lsof`, `Get-NetTCPConnection` | ★★★ | ★★★ |
| [dig-dns.md](dig-dns.md) | `dig`, `nslookup`, `host`, `Resolve-DnsName` | ★★★ | ★★★ |
| [ping-traceroute-mtr.md](ping-traceroute-mtr.md) | `ping`, `traceroute`, `mtr`, `nc`, `curl`, `Test-NetConnection` | ★★★ | ★★★ |
| [tcpdump-wireshark.md](tcpdump-wireshark.md) | `tcpdump`, Wireshark, `tshark` | ★★★ | ★★☆ |
| [ssh.md](ssh.md) | `ssh`, `scp`, `rsync`, `tmux`, Jump-Hosts | ★★★ | ★★★ |
| [git.md](git.md) | Git, GitLab/GitHub | ★★★ | ★★★ |
| [ansible.md](ansible.md) | Ansible | ★★★ | ★★★ |
| [monitoring-nagios.md](monitoring-nagios.md) | Nagios, Icinga, Checkmk, Zabbix, Prometheus, Grafana | ★★★ | ★★★ |
| [bash-scripting.md](bash-scripting.md) | Bash | ★★☆ | ★★★ |
| [powershell.md](powershell.md) | PowerShell | ★☆☆ | ★★★ |

★★★ = Pflicht · ★★☆ = wichtig · ★☆☆ = nützlich

## Lernreihenfolge

1. **Woche 1–4:** ping/traceroute, ss/netstat, dig, ssh
2. **Monat 2–3:** tcpdump/Wireshark, Bash-Grundlagen, PowerShell-Grundlagen
3. **Monat 3–6:** Git (ab hier alle eigenen Skripte versionieren!)
4. **2. Ausbildungsjahr:** Monitoring (Nagios-Konzept → Tool deines Betriebs), Ansible
5. **3. Ausbildungsjahr:** Ansible-Rollen, CI, Python
