# Ansible – Automatisierung

## Muss ich das können?

**Ja – sehr empfehlenswert, für beide Abteilungen.** Ansible ist der De-facto-Standard, um **Server und Netzwerkgeräte** gleichermaßen zu automatisieren:
- agentenlos (nur SSH bzw. WinRM / Geräte-API)
- YAML statt Programmiersprache → schneller Einstieg
- riesige Modul-Auswahl (Linux, Windows, Cisco, Arista, Juniper, Fortinet, VMware, Proxmox, NetBox …)

Ein Azubi, der Ansible beherrscht, ist im RZ sofort wertvoll. Voraussetzungen: Linux-Grundlagen, SSH, YAML, Git.

## Konzepte

| Begriff | Bedeutung |
|---|---|
| Control Node | Rechner, auf dem Ansible läuft (Linux) |
| Managed Node | Zielsystem |
| **Inventory** | Liste der Hosts und Gruppen |
| **Modul** | Einheit, die etwas tut (`apt`, `copy`, `service`, `ios_config` …) |
| **Task** | Aufruf eines Moduls |
| **Play** | Tasks für eine Hostgruppe |
| **Playbook** | YAML-Datei mit einem oder mehreren Plays |
| **Role** | wiederverwendbares Paket (Tasks, Templates, Variablen, Handler) |
| **Collection** | Distributionsformat für Module/Rollen (`ansible.posix`, `cisco.ios`) |
| **Handler** | Task, der nur bei Änderung läuft (z. B. Dienst neu starten) |
| **Facts** | automatisch gesammelte Infos über den Host |
| **Idempotenz** | Mehrfaches Ausführen ändert nichts, wenn Soll = Ist (→ `ok` statt `changed`) |

## Installation & Struktur

```bash
python3 -m venv ~/venv-ansible && source ~/venv-ansible/bin/activate
pip install ansible ansible-lint
ansible --version
ansible-galaxy collection install cisco.ios arista.eos community.general ansible.posix
```
```
ansible-projekt/
├── ansible.cfg
├── inventory/
│   ├── hosts.yml
│   ├── group_vars/
│   │   ├── all.yml
│   │   └── linux.yml
│   └── host_vars/
│       └── srv-web01.yml
├── playbooks/
│   └── baseline.yml
└── roles/
    └── baseline/
        ├── tasks/main.yml
        ├── handlers/main.yml
        ├── templates/chrony.conf.j2
        └── defaults/main.yml
```

## Inventory

```yaml
# inventory/hosts.yml
all:
  children:
    linux:
      hosts:
        srv-web01: { ansible_host: 10.20.0.11 }
        srv-web02: { ansible_host: 10.20.0.12 }
        srv-db01:  { ansible_host: 10.20.0.21 }
    leafs:
      hosts:
        leaf01: { ansible_host: 10.99.0.11 }
        leaf02: { ansible_host: 10.99.0.12 }
      vars:
        ansible_connection: ansible.netcommon.network_cli
        ansible_network_os: cisco.ios.ios
        ansible_user: netadmin
```
```bash
ansible-inventory -i inventory --graph
```

## Ad-hoc-Befehle

```bash
ansible linux -i inventory -m ping
ansible linux -i inventory -a "uptime"
ansible linux -i inventory -m ansible.builtin.setup -a "filter=ansible_distribution*"
ansible linux -i inventory -b -m ansible.builtin.package -a "name=htop state=present"   # -b = become (sudo)
ansible linux -i inventory -b -m ansible.builtin.service -a "name=chronyd state=restarted"
ansible leafs -i inventory -m cisco.ios.ios_command -a "commands='show version'"
```

## Playbook-Beispiel: Linux-Baseline

```yaml
# playbooks/baseline.yml
- name: Linux-Baseline
  hosts: linux
  become: true
  vars:
    ntp_servers: [10.0.0.123, 10.0.1.123]
    admin_users: [max, anna]
  tasks:
    - name: Pakete installieren
      ansible.builtin.package:
        name: [chrony, vim, htop, tmux, curl]
        state: present

    - name: Admin-Benutzer anlegen
      ansible.builtin.user:
        name: "{{ item }}"
        groups: wheel
        append: true
        shell: /bin/bash
      loop: "{{ admin_users }}"

    - name: SSH-Key hinterlegen
      ansible.posix.authorized_key:
        user: "{{ item }}"
        key: "{{ lookup('file', 'keys/' + item + '.pub') }}"
      loop: "{{ admin_users }}"

    - name: chrony konfigurieren
      ansible.builtin.template:
        src: templates/chrony.conf.j2
        dest: /etc/chrony.conf
        mode: "0644"
      notify: chrony neu starten

    - name: Root-Login per SSH verbieten
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        validate: 'sshd -t -f %s'
      notify: sshd neu laden

  handlers:
    - name: chrony neu starten
      ansible.builtin.service: { name: chronyd, state: restarted }
    - name: sshd neu laden
      ansible.builtin.service: { name: sshd, state: reloaded }
```
```jinja
{# templates/chrony.conf.j2 #}
{% for s in ntp_servers %}
server {{ s }} iburst
{% endfor %}
driftfile /var/lib/chrony/drift
makestep 1.0 3
```

## Playbook-Beispiel: Netzwerk

```yaml
- name: Baseline für Leaf-Switches
  hosts: leafs
  gather_facts: false
  tasks:
    - name: NTP, Syslog, Banner
      cisco.ios.ios_config:
        lines:
          - ntp server 10.0.0.123
          - logging host 10.0.0.514
          - service timestamps log datetime msec
        save_when: modified

    - name: Server-Port konfigurieren
      cisco.ios.ios_interfaces:
        config:
          - name: GigabitEthernet1/0/12
            description: "srv-web01 eth0"
            enabled: true
        state: merged

    - name: Version auslesen
      cisco.ios.ios_command:
        commands: show version
      register: ver

    - name: Ausgabe anzeigen
      ansible.builtin.debug:
        var: ver.stdout_lines[0][:3]
```

## Ausführen – sicher!

```bash
ansible-playbook -i inventory playbooks/baseline.yml --syntax-check
ansible-lint playbooks/baseline.yml
ansible-playbook -i inventory playbooks/baseline.yml --check --diff     # Trockenlauf: was WÜRDE sich ändern?
ansible-playbook -i inventory playbooks/baseline.yml --limit srv-web01  # erst ein Host
ansible-playbook -i inventory playbooks/baseline.yml                    # dann alle
ansible-playbook … --tags ssh ; --start-at-task "chrony konfigurieren" ; -v / -vvv
```
> Reihenfolge im Betrieb: **Syntax → Lint → `--check --diff` → ein Host → Gruppe**. Und das Ganze als Change mit Ticket.

## Fortgeschritten

- [ ] Variablen-Priorität (extra vars > host_vars > group_vars > role defaults)
- [ ] `when`-Bedingungen, `loop`, `register`, `failed_when`, `changed_when`, `block`/`rescue`
- [ ] **Ansible Vault** für Passwörter: `ansible-vault create secrets.yml`, `--ask-vault-pass`
- [ ] Rollen erstellen (`ansible-galaxy role init baseline`), Rollen aus Galaxy nutzen
- [ ] Dynamische Inventories (NetBox, VMware, Proxmox)
- [ ] Windows: `ansible.windows`-Collection über WinRM (`win_feature`, `win_service`, `win_updates`)
- [ ] **AWX / Ansible Automation Platform** (Web-UI, RBAC, Zeitpläne, Logs), Semaphore als leichte Alternative
- [ ] Testen mit Molecule

## Übungen

- [ ] Control Node + 3 Linux-VMs im Labor; Inventory mit Gruppen
- [ ] Baseline-Playbook (oben) umsetzen, zweimal laufen lassen – zweiter Lauf muss überall `ok` (nicht `changed`) zeigen
- [ ] Daraus eine Rolle machen und in Git versionieren
- [ ] Playbook, das Nginx mit Template-Konfiguration ausrollt und den Dienst bei Änderung neu lädt
- [ ] Netzwerk: 2 virtuelle Switches (cEOS/IOSv) per Ansible konfigurieren; Compliance-Check per `--check`
- [ ] Ansible Vault für ein Datenbankpasswort nutzen
- [ ] Zertifikat (später): Red Hat RHCE (EX294) ist eine reine Ansible-Prüfung
