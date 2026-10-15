# Git – Versionsverwaltung

## Muss ich das können?

**Ja – Pflicht, für beide Abteilungen.** Im modernen RZ liegen in Git:
- Ansible-Playbooks, Terraform-Code, Skripte (Bash, PowerShell, Python)
- **Netzwerkkonfigurationen** (automatisch per Oxidized/RANCID gesichert → `git diff` zeigt jede Änderung)
- Dokumentation (wie dieses Repo!), Dockerfiles, Kubernetes-Manifeste, Monitoring-Konfigurationen

Ohne Git kein Infrastructure as Code, keine Nachvollziehbarkeit („Wer hat gestern die ACL geändert?“), kein Rollback.

## Konzepte

- [ ] **Repository**, **Commit** (Schnappschuss mit Nachricht, Autor, Hash), **Branch**, **Merge**, **Remote** (GitLab/GitHub/Gitea), **Clone**
- [ ] Die drei Bereiche: **Working Directory** → `git add` → **Staging Area** → `git commit` → **Repository** → `git push` → **Remote**
- [ ] `HEAD`, `origin`, `main`/`master`
- [ ] Merge Request / Pull Request + Code-Review (Vier-Augen-Prinzip für Infrastruktur-Changes!)

## Einrichtung

```bash
git config --global user.name "Max Kudrasov"
git config --global user.email "max@example.com"
git config --global init.defaultBranch main
git config --global core.editor vim
git config --global pull.rebase false
git config --list
```

## Tägliche Befehle

```bash
git clone git@gitlab.firma.de:netz/ansible.git
git status                         # was ist geändert? (ständig nutzen)
git diff                           # Änderungen im Working Directory
git diff --staged                  # Änderungen in der Staging Area
git add datei.yml ; git add -p     # -p: Änderungen stückweise auswählen
git commit -m "leaf01: VLAN 120 für Kunde X hinzugefügt (CHG-4711)"
git push
git pull                           # = fetch + merge
git fetch                          # nur holen, nicht mergen
git log --oneline --graph --all
git log -p -- datei.cfg            # Änderungshistorie einer Datei inkl. Diffs
git log --since="2 days ago" --author="max"
git show <hash>                    # einen Commit anzeigen
git blame datei.cfg                # wer hat welche Zeile zuletzt geändert?
```

## Branches

```bash
git switch -c feature/ntp-baseline     # neuen Branch anlegen und wechseln
git branch -a                          # alle Branches
git switch main
git merge feature/ntp-baseline
git branch -d feature/ntp-baseline
git push -u origin feature/ntp-baseline   # Branch hochladen → Merge Request erstellen
```

## Fehler korrigieren

```bash
git restore datei.yml              # lokale Änderung verwerfen
git restore --staged datei.yml     # aus Staging nehmen
git commit --amend                 # letzten (noch nicht gepushten!) Commit korrigieren
git revert <hash>                  # Commit rückgängig machen per neuem Commit (sicher, auch nach Push)
git reset --soft HEAD~1            # letzten Commit aufheben, Änderungen behalten (nur lokal!)
git stash ; git stash pop          # Änderungen kurz beiseitelegen
git reflog                         # „Rettungsanker“ – alles, was HEAD je war
```
> Faustregel: Was schon **gepusht** ist und andere nutzen, nur mit `git revert` zurücknehmen – nie `reset --hard` + `push --force` auf geteilten Branches.

## Merge-Konflikte

```
<<<<<<< HEAD
ntp server 10.0.0.1
=======
ntp server 10.0.0.2
>>>>>>> feature/ntp
```
Datei bearbeiten (gewünschten Stand herstellen, Markierungen entfernen) → `git add datei` → `git commit`.

## .gitignore & Geheimnisse

```
# .gitignore
*.retry
.vault_pass
*.pem
*.key
.env
__pycache__/
.terraform/
*.tfstate*
```
- [ ] **Niemals Passwörter, private Schlüssel, Tokens committen** – auch nicht „kurz“. Einmal gepusht = kompromittiert (Historie!). Stattdessen: Ansible Vault, HashiCorp Vault, CI-Variablen.
- [ ] Tools: `gitleaks`, `trufflehog`, Pre-Commit-Hooks

## Gute Commit-Nachrichten

- Kurz, im Imperativ/Präsens, *was* und *warum*, Ticketnummer
- ✅ `firewall: Regel für Monitoring auf 10.50.0.0/24 ergänzt (INC-1234)`
- ❌ `fix`, `änderungen`, `asdf`

## Git im RZ-Alltag

- **Oxidized**: sichert stündlich alle Switch-Konfigs ins Git → `git log -p switches/leaf01` zeigt jede Änderung mit Zeitstempel
- **Ansible-Repo**: Änderung im Branch → Merge Request → Review durch Kollegen → CI prüft (ansible-lint) → Merge → Deployment
- **Doku**: Runbooks als Markdown im Repo, Wiki-Alternative
- **GitOps**: Kubernetes-Stand = Inhalt des Repos (Argo CD)

## SSH-Keys für Git

```bash
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub      # → in GitLab/GitHub unter SSH Keys eintragen
ssh -T git@github.com
```

## Übungen

- [ ] Dieses Repo klonen, nach jeder Lerneinheit Häkchen setzen und committen
- [ ] Branch anlegen, Notizen ergänzen, Merge Request erstellen und mergen
- [ ] Absichtlich einen Merge-Konflikt erzeugen und lösen
- [ ] Labor: Konfigurationen deiner Labor-Switches/Server in ein Git-Repo, Änderung machen, mit `git diff` nachvollziehen
- [ ] `git revert` und `git reflog` ausprobieren
- [ ] Interaktiv lernen: learngitbranching.js.org
