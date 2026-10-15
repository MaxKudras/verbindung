# Server Phase 2 – Linux-Server-Administration

**Dauer:** 8–10 Wochen (wichtigster Block!) · **Voraussetzung:** Linux-Grundlagen aus [00-fundament](../00-fundament.md)

Distributionen: **RHEL-Familie** (RHEL, Rocky, Alma) **und** **Debian/Ubuntu** – beide lernen. Befehle unten nennen beide Varianten, wo sie sich unterscheiden.

---

## 1. Installation & Boot

- [ ] Manuelle Installation, Partitionierung (`/boot`, `/boot/efi`, LVM für `/`, `/var`, `/home`, swap)
- [ ] Automatisiert: **Kickstart** (RHEL), **Preseed** / **Autoinstall/cloud-init** (Ubuntu), PXE-Server (DHCP + TFTP/HTTP)
- [ ] Bootablauf: UEFI → **GRUB2** → Kernel + **initramfs** → **systemd** (PID 1) → Targets
- [ ] GRUB: `/etc/default/grub`, `grub2-mkconfig -o …` / `update-grub`, Kernel-Parameter temporär beim Boot ändern (`e`)
- [ ] Targets: `multi-user.target`, `graphical.target`, `rescue.target`, `emergency.target`
- [ ] **Root-Passwort zurücksetzen** (`rd.break` bzw. `init=/bin/bash`) – klassische RHCSA-Aufgabe
- [ ] Kernel: `uname -r`, Module `lsmod`, `modprobe`, `modinfo`; alte Kernel entfernen

## 2. systemd

```bash
systemctl status nginx
systemctl start|stop|restart|reload nginx
systemctl enable --now nginx           # beim Boot + sofort
systemctl disable nginx
systemctl is-active nginx ; systemctl is-enabled nginx
systemctl list-units --type=service --state=running
systemctl --failed
systemctl cat nginx                     # Unit-Datei anzeigen
systemctl edit nginx                    # Override (drop-in) anlegen
systemctl daemon-reload                 # nach Unit-Änderung
systemctl get-default ; systemctl set-default multi-user.target
systemctl list-timers
systemd-analyze blame                   # was bremst den Boot?
hostnamectl set-hostname srv-web01
timedatectl set-timezone Europe/Berlin
```
- [ ] Eigene **Service-Unit** schreiben (`[Unit]`, `[Service]` mit `ExecStart`, `Restart=on-failure`, `User=`, `[Install]`)
- [ ] **systemd-Timer** als Cron-Alternative

### Logs mit journald
```bash
journalctl -u nginx                 # Logs eines Dienstes
journalctl -u nginx -f              # live
journalctl -b                       # seit letztem Boot
journalctl -b -1 -p err             # Fehler des vorherigen Boots (nach Absturz!)
journalctl --since "2026-10-15 08:00" --until "09:00"
journalctl -k                       # Kernel (wie dmesg)
journalctl --disk-usage ; journalctl --vacuum-time=14d
```
- [ ] Klassische Logs: `/var/log/messages` (RHEL), `/var/log/syslog` (Debian), `/var/log/secure` bzw. `/var/log/auth.log`, `/var/log/dmesg`, Dienst-Logs (`/var/log/nginx/`)
- [ ] **logrotate** (`/etc/logrotate.d/`), rsyslog-Weiterleitung an zentralen Server

## 3. Paketverwaltung

| Aufgabe | Debian/Ubuntu | RHEL/Rocky/Alma |
|---|---|---|
| Paketlisten aktualisieren | `apt update` | (automatisch) `dnf makecache` |
| Alles aktualisieren | `apt upgrade` / `apt full-upgrade` | `dnf upgrade` |
| Nur Sicherheitsupdates | `unattended-upgrades` | `dnf upgrade --security` |
| Installieren | `apt install nginx` | `dnf install nginx` |
| Entfernen | `apt remove` / `apt purge` | `dnf remove` |
| Suchen | `apt search` | `dnf search` |
| Infos | `apt show` | `dnf info` |
| Welches Paket enthält Datei? | `dpkg -S /usr/bin/dig` | `rpm -qf /usr/bin/dig` / `dnf provides dig` |
| Installierte Pakete | `dpkg -l` / `apt list --installed` | `rpm -qa` |
| Dateien eines Pakets | `dpkg -L paket` | `rpm -ql paket` |
| Historie / Rollback | `/var/log/apt/history.log` | `dnf history` / `dnf history undo N` |
| Repos | `/etc/apt/sources.list.d/` | `/etc/yum.repos.d/` |

- [ ] GPG-Schlüssel von Repos, interne Spiegel (Pulp, Satellite, Foreman/Katello, aptly)
- [ ] Neustart nötig? `needs-restarting -r` (RHEL), `/var/run/reboot-required` (Ubuntu)
- [ ] Snap/Flatpak kennen, auf Servern meist meiden
- [ ] Aus Quellen bauen (`./configure && make && make install`) – Grundzüge

## 4. Storage

```bash
lsblk -f                        # Blockgeräte + Dateisysteme
blkid
df -hT                          # Belegung + Typ
df -i                           # Inodes (voll trotz freiem Platz?!)
du -xh --max-depth=1 /var | sort -h
fdisk -l ; gdisk /dev/sdb ; parted /dev/sdb print
mkfs.xfs /dev/vg01/data ; mkfs.ext4 /dev/sdb1
mount /dev/vg01/data /data ; umount /data
findmnt
mount -a                        # fstab testen – VOR dem Reboot!
```
- [ ] `/etc/fstab` mit **UUID**, Optionen (`defaults`, `noatime`, `nofail`, `_netdev` für Netzlaufwerke)
- [ ] Dateisysteme: **XFS** (RHEL-Standard, nur vergrößerbar), **ext4** (verkleinerbar offline), Btrfs, ZFS
- [ ] **LVM** – Physical Volume → Volume Group → Logical Volume:
```bash
pvcreate /dev/sdb
vgcreate vg01 /dev/sdb
lvcreate -L 50G -n data vg01
lvextend -r -L +20G /dev/vg01/data     # -r vergrößert gleich das Dateisystem
vgextend vg01 /dev/sdc                  # neue Platte hinzufügen
pvs ; vgs ; lvs ; lvdisplay
lvcreate -s -L 5G -n data_snap /dev/vg01/data   # LVM-Snapshot
```
- [ ] Neue virtuelle Platte ohne Reboot erkennen: `echo "- - -" > /sys/class/scsi_host/host0/scan`; vergrößerte Platte: `echo 1 > /sys/class/block/sdb/device/rescan` + `growpart`
- [ ] Swap: `mkswap`, `swapon -s`, Swapfile
- [ ] Software-RAID: `mdadm --create`, `mdadm --detail /dev/md0`, `cat /proc/mdstat`
- [ ] Netzwerk-Storage: NFS-Client (`mount -t nfs`), iSCSI (`iscsiadm`), Multipath (`multipath -ll`) → Phase 5
- [ ] Verschlüsselung: LUKS (`cryptsetup`)
- [ ] Quotas (Grundzüge)

## 5. Netzwerk auf dem Server

```bash
ip -br a ; ip r ; ip -6 r
nmcli device status
nmcli connection show
nmcli con mod eth0 ipv4.addresses 10.110.0.10/24 ipv4.gateway 10.110.0.1 ipv4.dns "10.0.0.53" ipv4.method manual
nmcli con up eth0
nmtui                                   # Text-Oberfläche
cat /etc/netplan/*.yaml ; netplan try ; netplan apply    # Ubuntu
resolvectl status                       # systemd-resolved
cat /etc/resolv.conf /etc/hosts /etc/nsswitch.conf
ss -tulpn                               # siehe tools/netstat-ss.md
```
- [ ] **Bonding** (Modi: 0 balance-rr, **1 active-backup**, **4 802.3ad/LACP**, 6 balance-alb) – Gegenstück zum Port-Channel/MLAG
- [ ] VLAN-Interfaces (`eth0.110`), Bridges (für KVM)
- [ ] Statische Routen persistent
- [ ] Namensauflösung: Reihenfolge in `/etc/nsswitch.conf` (files dns)
- [ ] MTU setzen (Jumbo für Storage)

## 6. Benutzer, Rechte, Authentifizierung

- [ ] Lokale Benutzer/Gruppen (Fundament), `chage` (Passwortalter), `/etc/login.defs`
- [ ] **sudo** granular (`/etc/sudoers.d/`), Logging von sudo
- [ ] PAM-Grundzüge (`/etc/pam.d/`)
- [ ] Zentrale Authentifizierung: **SSSD** + LDAP / FreeIPA / **Active Directory** (`realm join firma.local`)
- [ ] ACLs, `umask`, Sonderrechte

## 7. SSH

> Tiefgang: [`tools/ssh.md`](../tools/ssh.md)

- [ ] Schlüsselbasierte Anmeldung (`ed25519`), `authorized_keys`, Agent
- [ ] `sshd_config` härten: `PermitRootLogin no`, `PasswordAuthentication no`, `AllowGroups`
- [ ] Jump-Host / Bastion, `~/.ssh/config`
- [ ] Dateien übertragen: `scp`, `sftp`, **`rsync -avzP`**

## 8. Prozesse & Performance

```bash
ps aux --sort=-%mem | head
ps -ef --forest
pstree -p
top / htop                       # Load, %CPU, %MEM, wa (IO-Wait!), st (Steal bei VMs!)
uptime                           # Load Average 1/5/15 min – im Verhältnis zur Kernzahl lesen!
nproc
free -h                          # "available" ist entscheidend, nicht "free"
vmstat 1 5                       # r, b, si/so (Swapping!), wa
iostat -xz 1                     # %util, await pro Platte (Paket sysstat)
sar -u -r -d 1 5                 # historische Werte: sar -f /var/log/sa/saXX
iotop ; iftop ; nload ; nethogs
pidstat 1
lsof -p PID ; lsof -i :443 ; lsof +L1   # gelöschte, aber offene Dateien (Platte "voll"?)
kill PID ; kill -9 PID ; pkill -f muster ; killall
nice -n 10 befehl ; renice +5 -p PID
strace -p PID -f -e trace=network   # was macht der Prozess?
dmesg -T | tail                  # OOM-Killer? Hardwarefehler?
ulimit -a
```
- [ ] **Load Average** verstehen (laufende + auf IO wartende Prozesse)
- [ ] **OOM-Killer** erkennen (`dmesg | grep -i oom`)
- [ ] Signale: SIGTERM (15), SIGKILL (9), SIGHUP (1, oft „Konfig neu laden“)
- [ ] Performance-Methode: **USE** – Utilization, Saturation, Errors für CPU, RAM, Disk, Netz
- [ ] `/proc` erkunden: `/proc/cpuinfo`, `/proc/meminfo`, `/proc/PID/`
- [ ] Kernel-Parameter: `sysctl -a`, `/etc/sysctl.d/`

## 9. Geplante Aufgaben

- [ ] **cron**: `crontab -e`, `crontab -l`, `/etc/cron.d/`, `/etc/cron.daily/`
```
# ┌ Minute (0-59)  ┌ Stunde (0-23)  ┌ Tag (1-31)  ┌ Monat (1-12)  ┌ Wochentag (0-7, 0/7 = So)
# m h dom mon dow  befehl
30 2 * * *   /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
*/5 * * * *  /usr/local/bin/check.sh
0 6 * * 1-5  /usr/local/bin/report.sh
```
- [ ] systemd-Timer (`OnCalendar=*-*-* 02:30:00`), `at` für einmalige Aufgaben
- [ ] crontab.guru zum Prüfen

## 10. Sicherheit & Härtung

- [ ] **SELinux** (RHEL): Modi enforcing/permissive/disabled, Kontexte, Booleans
```bash
getenforce ; sestatus
ls -Z /var/www/html
restorecon -Rv /var/www/html
semanage fcontext -a -t httpd_sys_content_t "/data/web(/.*)?"
setsebool -P httpd_can_network_connect on
ausearch -m avc -ts recent ; sealert -a /var/log/audit/audit.log
```
  **Nicht einfach abschalten** – Fehler lesen und richtig beheben!
- [ ] **AppArmor** (Ubuntu): `aa-status`
- [ ] Host-Firewall: firewalld / ufw / nftables (→ [Netzwerk Phase 5](../netzwerk/05-sicherheit.md))
- [ ] **fail2ban**, `auditd`, AIDE (Dateiintegrität)
- [ ] Unnötige Dienste und Pakete entfernen, offene Ports prüfen (`ss -tulpn`)
- [ ] Automatische Sicherheitsupdates, Patch-Management-Prozess
- [ ] **CIS Benchmarks**, BSI-Grundschutz-Bausteine SYS.1.1 / SYS.1.3, Lynis (`lynis audit system`)
- [ ] Zertifikate & TLS (`openssl`, Let's Encrypt mit certbot/acme.sh)

## 11. Dienste – jeden einmal selbst aufsetzen

- [ ] **Webserver**: Nginx und Apache (VirtualHosts / Server-Blocks, TLS, Reverse Proxy, Logs) – `nginx -t`, `apachectl configtest`
- [ ] **Datenbanken**: MariaDB/MySQL und PostgreSQL – Installation, Benutzer, Rechte, Dump/Restore (`mysqldump`, `pg_dump`), Replikation (Grundzüge)
- [ ] **NFS**-Server (`/etc/exports`, `exportfs -ra`, `showmount -e`)
- [ ] **Samba** (Freigaben für Windows, ggf. AD-Mitglied)
- [ ] **DNS** (BIND/Unbound) und **DHCP** (Kea) – siehe [Netzwerk Phase 4](../netzwerk/04-netzwerkdienste.md)
- [ ] **Mail** (Postfix als Relay für Systemmails) – Grundzüge
- [ ] **Proxy** (Squid) – Grundzüge
- [ ] **Zeit** (chrony)

## 12. Scripting

> Tiefgang: [`tools/bash-scripting.md`](../tools/bash-scripting.md)

- [ ] Bash-Skripte mit Variablen, Bedingungen, Schleifen, Funktionen, Exit-Codes, `set -euo pipefail`
- [ ] Python für komplexere Aufgaben

## 13. Troubleshooting-Checkliste Linux

- [ ] Läuft der Dienst? `systemctl status`, `journalctl -u … -e`
- [ ] Lauscht er? `ss -tulpn | grep :443`
- [ ] Firewall? `firewall-cmd --list-all` / `nft list ruleset`
- [ ] SELinux? `ausearch -m avc -ts recent`
- [ ] Platte voll? `df -h`, `df -i`, `lsof +L1`
- [ ] RAM/Swap? `free -h`, `dmesg | grep -i oom`
- [ ] CPU/IO? `top` (wa, st), `iostat -x`
- [ ] Netzwerk? `ip a`, `ip r`, `ping Gateway`, `dig`
- [ ] Zeit korrekt? `timedatectl`
- [ ] Was wurde geändert? `dnf history`, `/var/log/apt/history.log`, `last`, Git/Ansible-Logs

## 14. Praxisaufgaben

- [ ] Rocky Linux und Ubuntu Server installieren, beide nach Checkliste härten
- [ ] LVM: VG aus 2 Platten, LV anlegen, online vergrößern
- [ ] Nginx mit TLS als Reverse Proxy vor einer App; SELinux bleibt enforcing
- [ ] Eigene systemd-Unit + Timer für ein Backup-Skript
- [ ] Linux-Server ins AD joinen (`realm join`), Anmeldung mit AD-Benutzer
- [ ] PXE-/Kickstart-Server, der einen Rocky-Server vollautomatisch installiert
- [ ] RHCSA-Übungsprüfung durcharbeiten (z. B. aus Sander van Vugts Buch)

## Meine Notizen

