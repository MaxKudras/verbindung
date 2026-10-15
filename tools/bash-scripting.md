# Bash-Scripting

## Muss ich das können?

**Server: Pflicht. Netzwerk: wichtig.** Bash ist der schnellste Weg, wiederkehrende Aufgaben auf Linux zu automatisieren (Log-Auswertung, Backups, Checks, Massenänderungen). Für Größeres → Ansible oder Python.

## Grundgerüst

```bash
#!/usr/bin/env bash
# Zweck: …   Autor: …   Datum: …
set -euo pipefail          # bei Fehler abbrechen, undefinierte Variablen = Fehler, Fehler in Pipes erkennen
IFS=$'\n\t'

LOGFILE="/var/log/meinskript.log"
log() { echo "$(date '+%F %T') $*" | tee -a "$LOGFILE"; }

main() {
    log "Start"
    # …
    log "Ende"
}
main "$@"
```
```bash
chmod +x skript.sh ; ./skript.sh
bash -n skript.sh          # Syntax prüfen
bash -x skript.sh          # Debug: jede Zeile anzeigen
shellcheck skript.sh       # Linter – unbedingt nutzen!
```

## Bausteine

```bash
# Variablen
NAME="srv01"; echo "$NAME" ; echo "${NAME}_backup"
HEUTE=$(date +%F)                       # Befehlsersetzung
ZAHL=$(( 5 + 3 ))                       # Arithmetik
readonly CONFIG="/etc/app.conf"

# Argumente
echo "$0 $1 $2 $# $@"                   # Skriptname, Arg1, Arg2, Anzahl, alle
[ $# -lt 1 ] && { echo "Usage: $0 <host>"; exit 1; }

# Bedingungen
if [[ -f /etc/os-release ]]; then … elif …; else …; fi
[[ -d dir ]]  [[ -z "$var" ]]  [[ -n "$var" ]]  [[ "$a" == "$b" ]]  [[ $x -gt 5 ]]  [[ "$s" =~ ^srv ]]
case "$1" in
  start) … ;;
  stop)  … ;;
  *) echo "unbekannt"; exit 1 ;;
esac

# Schleifen
for h in srv01 srv02 srv03; do ping -c1 -W1 "$h" >/dev/null && echo "$h up" || echo "$h DOWN"; done
for i in {1..254}; do ping -c1 -W1 10.0.0.$i &>/dev/null && echo "10.0.0.$i" & done; wait
while read -r host ip; do echo "$host → $ip"; done < hosts.txt
while true; do date; sleep 5; done

# Funktionen + Rückgabewert
check_port() { timeout 2 bash -c "</dev/tcp/$1/$2" 2>/dev/null; }
if check_port 10.0.0.5 443; then echo offen; fi

# Exit-Codes
befehl; echo $?                         # 0 = Erfolg
befehl && echo ok || echo fehler

# Arrays
SERVER=(srv01 srv02 srv03); echo "${SERVER[@]}" "${#SERVER[@]}"

# Here-Doc
cat > /etc/motd <<EOF
Server: $(hostname) – verwaltet durch Ansible
EOF

# Aufräumen bei Abbruch
TMP=$(mktemp); trap 'rm -f "$TMP"' EXIT
```

## Praxisbeispiele

```bash
# 1) Plattenplatz-Warnung
#!/usr/bin/env bash
set -euo pipefail
LIMIT=85
df -P -x tmpfs -x devtmpfs | awk 'NR>1 {gsub("%","",$5); print $5, $6}' | while read -r use mnt; do
  [ "$use" -ge "$LIMIT" ] && echo "WARNUNG: $mnt zu ${use}% voll auf $(hostname)"
done

# 2) Fehlgeschlagene SSH-Logins pro IP
grep "Failed password" /var/log/auth.log | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort | uniq -c | sort -rn | head

# 3) Einfaches Backup mit Rotation
#!/usr/bin/env bash
set -euo pipefail
ZIEL=/backup; QUELLE=/etc
tar -czf "$ZIEL/etc-$(hostname)-$(date +%F).tar.gz" "$QUELLE"
find "$ZIEL" -name 'etc-*.tar.gz' -mtime +14 -delete

# 4) Ist ein Zertifikat bald abgelaufen?
host=$1; end=$(echo | openssl s_client -connect "$host:443" -servername "$host" 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
days=$(( ( $(date -d "$end" +%s) - $(date +%s) ) / 86400 )); echo "$host: noch $days Tage"
```

## Übungen

- [ ] Skript, das für eine Liste von Hosts Ping + SSH-Port prüft und eine Tabelle ausgibt
- [ ] Log-Analyse: Top-10-URLs und HTTP-Statuscodes aus einem Nginx-Log
- [ ] Backup-Skript mit Logging, Rotation, Exit-Codes – per systemd-Timer ausführen
- [ ] Jedes Skript mit `shellcheck` fehlerfrei bekommen und in Git ablegen
- [ ] Ressourcen: „The Linux Command Line“ (William Shotts), OverTheWire Bandit, Bash-Hackers-Wiki
