# Server Phase 6 – Container, Automatisierung & Betrieb

**Dauer:** laufend (ab 2. Ausbildungsjahr) · **Voraussetzung:** Phase 2 (Linux)

---

## 1. Container mit Docker / Podman

- [ ] Container vs. VM (gemeinsamer Kernel, Namespaces, cgroups)
- [ ] Image, Container, Registry (Docker Hub, Harbor, GitLab Registry), Tags (nie blind `latest` in Produktion)
- [ ] **Podman** (rootless, daemonlos, RHEL-Standard) – Befehle fast identisch zu Docker
```bash
docker run -d --name web -p 8080:80 -v /srv/web:/usr/share/nginx/html:ro nginx:1.27
docker ps -a
docker logs -f web
docker exec -it web sh
docker inspect web
docker stats
docker stop web ; docker rm web
docker images ; docker pull ; docker rmi
docker build -t meinapp:1.0 .
docker volume ls ; docker network ls
docker system df ; docker system prune
```
- [ ] **Dockerfile** schreiben (`FROM`, `RUN`, `COPY`, `ENV`, `EXPOSE`, `USER`, `CMD`/`ENTRYPOINT`), Multi-Stage-Builds, kleine Base-Images
- [ ] **Docker Compose** (mehrere Container, Netzwerke, Volumes, `.env`)
```bash
docker compose up -d ; docker compose ps ; docker compose logs -f ; docker compose down
```
- [ ] Sicherheit: nicht als root im Container, Images scannen (Trivy), Secrets nicht ins Image

## 2. Kubernetes (Grundzüge)

- [ ] Architektur: Control Plane (API-Server, etcd, Scheduler, Controller Manager), Worker Nodes (kubelet, kube-proxy, Container Runtime)
- [ ] Objekte: **Pod**, **Deployment**, ReplicaSet, **Service** (ClusterIP, NodePort, LoadBalancer), **Ingress**, ConfigMap, Secret, PersistentVolume(Claim), Namespace, StatefulSet, DaemonSet
- [ ] Distributionen: k3s (Labor), RKE2, OpenShift, Rancher, Managed (EKS/AKS/GKE)
```bash
kubectl get nodes
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> [-f] [--previous]
kubectl exec -it <pod> -- sh
kubectl apply -f deployment.yaml
kubectl rollout status deployment/web ; kubectl rollout undo deployment/web
kubectl scale deployment web --replicas=3
kubectl get events --sort-by=.lastTimestamp
```
- [ ] Typische Pod-Fehler: `CrashLoopBackOff`, `ImagePullBackOff`, `Pending` (keine Ressourcen), `OOMKilled`
- [ ] Helm (Paketmanager), GitOps (Argo CD, Flux) – Ausblick
- [ ] Zertifikat (später): CKA

## 3. Konfigurationsmanagement mit Ansible

> Tiefgang: [`tools/ansible.md`](../tools/ansible.md)

- [ ] Inventory, Ad-hoc-Befehle, Playbooks, Rollen, Variablen, Templates (Jinja2), Handlers, Vault
- [ ] Idempotenz verstehen
- [ ] Typische Server-Aufgaben: Benutzer + SSH-Keys verteilen, Pakete/Updates, Härtung, Dienste konfigurieren, NTP/DNS/Syslog einheitlich setzen
- [ ] Alternativen kennen: Puppet, Chef, SaltStack; zentrale Plattformen: AWX / Ansible Automation Platform, Foreman, Semaphore

## 4. Infrastructure as Code

- [ ] **Terraform / OpenTofu**: Provider (vSphere, Proxmox, Cloud, NetBox), Ressourcen, State-Datei, `terraform init / plan / apply / destroy`
- [ ] **Packer**: Golden Images/Templates automatisiert bauen
- [ ] **cloud-init**: Erstkonfiguration von VMs (Hostname, Benutzer, SSH-Keys, Pakete)
- [ ] Prinzip: deklarativ, versioniert in **Git**, Review per Merge Request, reproduzierbar

## 5. Versionsverwaltung & CI/CD

> Tiefgang: [`tools/git.md`](../tools/git.md)

- [ ] Skripte, Playbooks, Terraform, Dockerfiles, Doku → alles in Git (GitLab/GitHub/Gitea)
- [ ] CI/CD-Pipelines (GitLab CI, GitHub Actions, Jenkins): Lint (ansible-lint, shellcheck, yamllint) → Test → Deploy

## 6. Monitoring, Logging, Alarmierung

> Tiefgang: [`tools/monitoring-nagios.md`](../tools/monitoring-nagios.md)

- [ ] Server-Metriken: CPU, RAM, Disk (Füllstand + Prognose!), IO, Netz, Dienste, Prozesse, Zertifikate, Backupstatus, Hardware (über OOB/Redfish)
- [ ] Werkzeuge: **Nagios/Icinga/Checkmk**, **Zabbix**, **Prometheus + node_exporter + Alertmanager + Grafana**
- [ ] Logs zentral: **ELK/OpenSearch**, **Graylog**, **Loki + Grafana**, Splunk
- [ ] SLI/SLO, Dashboards sinnvoll gestalten (keine 100 bunten Graphen)
- [ ] On-Call/Rufbereitschaft, Runbooks pro Alarm

## 7. Härtung & Compliance

- [ ] **CIS Benchmarks** (Linux, Windows, Docker, Kubernetes)
- [ ] **BSI IT-Grundschutz**: Bausteine SYS.1.1 (Allgemeiner Server), SYS.1.3 (Linux), SYS.1.2 (Windows Server), SYS.1.5 (Virtualisierung), APP.4.4 (Kubernetes); Schutzbedarfsfeststellung
- [ ] ISO 27001-Grundzüge, NIS2 (betrifft viele RZ-Betreiber)
- [ ] Patch-Management-Prozess (Test → Staging → Produktion), CVE-Bewertung (CVSS)
- [ ] Schwachstellen-Scans: Greenbone/OpenVAS, Nessus, Lynis, Trivy
- [ ] Secrets-Management: HashiCorp Vault, Ansible Vault, Passwort-Manager (Keepass, Bitwarden/Vaultwarden, Passbolt)
- [ ] Least Privilege, MFA für Admin-Zugänge, Bastion/Jump-Host, Session-Recording

## 8. Praxisaufgaben

- [ ] Docker-Compose-Stack: Nginx + App + PostgreSQL mit persistenten Volumes; Backup der DB
- [ ] k3s auf 3 VMs, eine App mit Deployment + Service + Ingress ausrollen
- [ ] Ansible-Rolle „baseline“ (Benutzer, SSH-Härtung, NTP, Firewall, node_exporter) für alle Labor-Linux-Server
- [ ] Terraform: VMs auf Proxmox per Code anlegen, mit cloud-init + Ansible konfigurieren
- [ ] Prometheus + Grafana + Alertmanager: Alarm, wenn Platte > 85 % oder Dienst down
- [ ] Alles in einem Git-Repo mit Pipeline (ansible-lint, shellcheck)

## Meine Notizen

