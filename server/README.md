# Lernpfad Server

Ziel: Am Ende der Ausbildung Linux- und Windows-Server sowie Virtualisierungsplattformen im RZ **selbständig installieren, betreiben, absichern, sichern und Störungen beheben** – auf dem Niveau von LPIC-1/RHCSA plus solider Windows-Server- und Virtualisierungspraxis.

## Phasen

| Phase | Datei | Dauer | Kernthemen |
|---|---|---|---|
| 1 | [01-hardware-oob.md](01-hardware-oob.md) | 3–4 Wo. | Serverhardware, RAID, UEFI, iDRAC/iLO/IPMI/Redfish, RZ-Handwerk |
| 2 | [02-linux.md](02-linux.md) | 8–10 Wo. | Boot, systemd, Pakete, Storage/LVM, Netzwerk, SSH, Sicherheit, Dienste, Performance |
| 3 | [03-windows-server.md](03-windows-server.md) | 6–8 Wo. | AD, GPO, DNS/DHCP, Dateiserver, PowerShell, WSUS, PKI |
| 4 | [04-virtualisierung.md](04-virtualisierung.md) | 6–8 Wo. | vSphere, Proxmox, Hyper-V, KVM, Cluster, HA |
| 5 | [05-storage-backup.md](05-storage-backup.md) | 4–6 Wo. | SAN/NAS, iSCSI/FC/NFS, Ceph, Backup-Strategien, DR |
| 6 | [06-container-iac.md](06-container-iac.md) | laufend | Docker, Kubernetes, Ansible, Terraform, Git, Monitoring, Härtung |

## Werkzeuge, die dich durch den gesamten Pfad begleiten

- [`tools/ssh.md`](../tools/ssh.md) – SSH, Schlüssel, Jump-Hosts, rsync
- [`tools/netstat-ss.md`](../tools/netstat-ss.md) – Welcher Dienst lauscht wo?
- [`tools/dig-dns.md`](../tools/dig-dns.md) – DNS-Probleme auf Servern
- [`tools/bash-scripting.md`](../tools/bash-scripting.md) – Automatisieren auf Linux
- [`tools/powershell.md`](../tools/powershell.md) – Automatisieren auf Windows
- [`tools/git.md`](../tools/git.md) – Skripte & Konfigs versionieren
- [`tools/ansible.md`](../tools/ansible.md) – Server-Konfigurationsmanagement
- [`tools/monitoring-nagios.md`](../tools/monitoring-nagios.md) – Server überwachen

## Aufgabe: Bestandsaufnahme in deinem RZ

Notiere (ohne vertrauliche Details):

- Server-Hersteller: …
- OOB-Management: (iDRAC / iLO / XClarity / IPMI) …
- Linux-Distributionen: …
- Windows-Server-Versionen: …
- Hypervisor: …
- Storage-Systeme: …
- Backup-Software: …
- Monitoring: …
- Automatisierung (Ansible/Puppet/…): …
- Ticketsystem / CMDB: …

## Abschlussziele des Server-Pfads

- [ ] Ich kann einen neuen Server physisch einbauen, per OOB erreichbar machen, RAID konfigurieren und ein OS installieren (manuell **und** automatisiert per PXE/Template).
- [ ] Ich kann einen Linux-Server nach Härtungsrichtlinie aufsetzen, Dienste betreiben und Störungen über Logs/Metriken eingrenzen.
- [ ] Ich kann AD-Benutzer, Gruppen und GPOs nach AGDLP verwalten und AD-Replikationsprobleme diagnostizieren.
- [ ] Ich kann VMs auf der Virtualisierungsplattform anlegen, migrieren und einen Host-Ausfall erklären (HA).
- [ ] Ich kann einen Restore aus dem Backup durchführen – Datei, VM und Datenbank.
- [ ] Ich habe eine wiederkehrende Server-Aufgabe mit Ansible/Bash/PowerShell automatisiert und in Git versioniert.
