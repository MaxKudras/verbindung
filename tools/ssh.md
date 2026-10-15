# SSH, Dateiübertragung & tmux

## Muss ich das können?

**Ja – Pflicht.** SSH ist der Zugang zu praktisch jedem Linux-Server, Switch, Router und jeder Firewall im RZ.

## Grundlagen

- [ ] Protokoll SSH-2, Port 22, Host-Keys (`/etc/ssh/ssh_host_*`), `known_hosts` (Warnung „REMOTE HOST IDENTIFICATION HAS CHANGED“ ernst nehmen – Neuinstallation oder MITM?)
- [ ] Authentifizierung: Passwort, **Public Key**, Zertifikate (SSH-CA), Kerberos/GSSAPI, MFA

## Schlüssel

```bash
ssh-keygen -t ed25519 -C "max@laptop"          # Schlüsselpaar erzeugen (mit Passphrase!)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@srv01
cat ~/.ssh/authorized_keys                     # auf dem Server
chmod 700 ~/.ssh ; chmod 600 ~/.ssh/authorized_keys   # falsche Rechte = Key wird ignoriert!
eval "$(ssh-agent)" ; ssh-add                  # Agent, Passphrase nur einmal eingeben
ssh-keygen -lf ~/.ssh/id_ed25519.pub           # Fingerprint
ssh-keygen -R srv01                            # alten Host-Key aus known_hosts entfernen
```
**Privater Schlüssel verlässt nie deinen Rechner.** Nie in Git, nie per Mail.

## Verbinden

```bash
ssh user@srv01
ssh -p 2222 user@srv01
ssh -i ~/.ssh/kunde_ed25519 user@srv01
ssh -J jump.firma.de user@10.20.0.5            # über Jump-Host / Bastion
ssh -v user@srv01                              # Debug (bis -vvv) – bei Login-Problemen
ssh user@srv01 'uptime; df -h'                 # Befehl remote ausführen
ssh -o ConnectTimeout=5 user@srv01
for h in srv01 srv02 srv03; do ssh $h 'hostname; uptime'; done
```
Ältere Netzwerkgeräte brauchen manchmal alte Algorithmen:
`ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa admin@switch01`

## ~/.ssh/config – spart enorm viel Tipparbeit

```
Host jump
    HostName jump.firma.de
    User max
    IdentityFile ~/.ssh/id_ed25519

Host srv-*
    User max
    ProxyJump jump

Host srv-web01
    HostName 10.20.0.11

Host *
    ServerAliveInterval 30
    AddKeysToAgent yes
```
Dann einfach: `ssh srv-web01`

## Tunnel (Port-Forwarding)

```bash
ssh -L 8443:10.99.1.21:443 jump     # lokal :8443 → über jump → iDRAC 10.99.1.21:443 (Browser: https://localhost:8443)
ssh -R 9000:localhost:80 srv01      # Remote-Forward
ssh -D 1080 jump                    # SOCKS-Proxy über jump
ssh -N -f -L 5432:db01:5432 jump    # nur Tunnel, im Hintergrund
```

## Server härten (`/etc/ssh/sshd_config` bzw. `sshd_config.d/*.conf`)

```
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AllowGroups ssh-users
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
```
```bash
sshd -t                         # Syntax prüfen – IMMER vor dem Neustart!
systemctl reload sshd
```
> **Goldene Regel:** Bei Änderungen an sshd die **bestehende Sitzung offen lassen** und in einer **zweiten** Sitzung testen. Sonst sperrst du dich aus (dann hilft nur OOB-Konsole).

## Dateien übertragen

```bash
scp datei.txt user@srv01:/tmp/
scp -r ordner/ user@srv01:/tmp/
scp user@srv01:/var/log/syslog .
sftp user@srv01
rsync -avzP quelle/ user@srv01:/ziel/         # -a Archiv, -v, -z Kompression, -P Fortschritt + Fortsetzen
rsync -avz --delete quelle/ ziel/              # Spiegeln (Vorsicht: löscht im Ziel!)
rsync -avzn …                                  # -n = Dry Run, erst testen!
rsync -avz -e "ssh -J jump" quelle/ srv01:/ziel/
```
**Achtung Slash:** `rsync quelle/ ziel/` kopiert den *Inhalt*, `rsync quelle ziel/` den *Ordner selbst*.

## tmux – Sitzungen, die SSH-Abbrüche überleben

```bash
tmux new -s wartung          # neue Sitzung
# Ctrl+b d                   → abkoppeln (läuft weiter!)
tmux ls
tmux attach -t wartung
# Ctrl+b c neues Fenster, Ctrl+b n/p wechseln, Ctrl+b % / " teilen, Ctrl+b [ scrollen
```
Lange Updates, Datenmigrationen, `rsync` großer Datenmengen → **immer in tmux** (oder `screen`).

## Windows

- [ ] OpenSSH-Client ist in Windows integriert (`ssh` in PowerShell), PuTTY/KiTTY, MobaXterm, Windows Terminal
- [ ] WinSCP für Dateiübertragung
- [ ] OpenSSH-Server optional installierbar

## Übungen

- [ ] Ed25519-Key erzeugen, auf alle Labor-Server verteilen, Passwort-Login deaktivieren
- [ ] `~/.ssh/config` mit Jump-Host für dein Labor anlegen
- [ ] Per SSH-Tunnel auf die Web-GUI einer VM zugreifen, die nur intern erreichbar ist
- [ ] Großes Verzeichnis per rsync übertragen, Verbindung abbrechen, fortsetzen
