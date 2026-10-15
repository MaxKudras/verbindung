# dig & DNS-Werkzeuge

## Muss ich das können?

**Ja – Pflicht für beide Abteilungen.** Der Spruch „It's always DNS“ kommt nicht von ungefähr: Mailzustellung, AD-Anmeldung, Zertifikate, Webseiten, Kubernetes, Monitoring – alles hängt an DNS.

`dig` (Paket `bind9-dnsutils` / `bind-utils`) ist das Standard-Werkzeug, weil es **die rohe DNS-Antwort** zeigt (Flags, TTL, Sections, antwortender Server). `nslookup` gibt es überall (auch Windows), ist aber weniger aussagekräftig. Unter Windows ist `Resolve-DnsName` die moderne Alternative.

## Grundaufruf & Ausgabe lesen

```bash
dig www.example.com
```
```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4242
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;www.example.com.        IN  A

;; ANSWER SECTION:
www.example.com.   3600  IN  A  93.184.215.14

;; Query time: 12 msec
;; SERVER: 10.0.0.53#53(10.0.0.53) (UDP)
```
| Feld | Bedeutung |
|---|---|
| `status` | **NOERROR** (ok), **NXDOMAIN** (Name existiert nicht), **SERVFAIL** (Server konnte nicht auflösen – oft DNSSEC- oder Upstream-Problem), **REFUSED** (Server verweigert, z. B. keine Rekursion erlaubt) |
| `flags` | `qr` Antwort, `rd` Rekursion gewünscht, `ra` Rekursion verfügbar, **`aa` authoritative answer**, `ad` DNSSEC-validiert |
| `3600` | TTL in Sekunden (wie lange gecacht wird – bei Caching-Resolvern zählt sie herunter) |
| `SERVER` | **welcher** DNS-Server geantwortet hat |

## Die wichtigsten dig-Befehle

```bash
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig example.com SOA
dig example.com TXT              # SPF, DMARC, Domain-Verifikation
dig _dmarc.example.com TXT
dig selector1._domainkey.example.com TXT   # DKIM
dig example.com CAA
dig _ldap._tcp.dc._msdcs.firma.local SRV   # AD-Domänencontroller finden
dig example.com ANY              # (viele Server liefern das nicht mehr)

dig +short www.example.com       # nur das Ergebnis – ideal für Skripte
dig -x 203.0.113.10              # Reverse-Lookup (PTR)
dig @8.8.8.8 www.example.com     # bestimmten Server fragen
dig @ns1.example.com www.example.com +norecurse   # direkt den autoritativen fragen
dig +trace www.example.com       # gesamte Kette Root → TLD → autoritativ
dig +noall +answer www.example.com    # nur Answer-Section
dig +nocmd +noall +answer +ttlid example.com MX
dig +dnssec example.com          # DNSSEC-Signaturen mitliefern
dig +tcp example.com             # über TCP erzwingen
dig axfr firma.local @ns1.firma.local   # Zonentransfer (nur, wenn erlaubt)
dig -f namen.txt +short          # viele Namen aus Datei
```

## Andere Werkzeuge

```bash
host www.example.com
host -t MX example.com
host 203.0.113.10                 # reverse

nslookup www.example.com
nslookup -type=MX example.com 8.8.8.8
nslookup                          # interaktiv: server 8.8.8.8 / set type=SRV / …

resolvectl query www.example.com  # über systemd-resolved (zeigt, was das SYSTEM wirklich sieht)
resolvectl status                 # welcher DNS-Server pro Interface?
getent hosts www.example.com      # inkl. /etc/hosts (nsswitch) – das, was Programme sehen!
```
```powershell
Resolve-DnsName www.example.com
Resolve-DnsName example.com -Type MX -Server 8.8.8.8
Resolve-DnsName -Name 203.0.113.10 -Type PTR
Get-DnsClientServerAddress
Get-DnsClientCache ; Clear-DnsClientCache
ipconfig /displaydns ; ipconfig /flushdns
```

> **Wichtig:** `dig` und `nslookup` fragen DNS **direkt** und ignorieren `/etc/hosts`. Programme nutzen dagegen die System-Auflösung (`getent hosts`). Wenn `dig` richtig antwortet, die Anwendung aber nicht, prüfe `/etc/hosts`, `/etc/nsswitch.conf` und lokale Caches!

## Diagnose-Workflow „Name wird nicht / falsch aufgelöst“

1. `getent hosts name` – was sieht das System? (`/etc/hosts`?)
2. `dig name` – was sagt der konfigurierte Resolver? Status? TTL?
3. `dig @<anderer-resolver> name` – liegt es am Resolver (Cache, Forwarder)?
4. `dig name NS` + `dig @<autoritativer-ns> name +norecurse` – was steht **wirklich** in der Zone? `aa`-Flag?
5. `dig +trace name` – wo bricht die Delegationskette ab?
6. `dig name SOA` auf allen Nameservern – stimmt die **Serial** überall (Replikation/Zonentransfer ok)?
7. Bei SERVFAIL: `dig +cd name` (Checking Disabled) – funktioniert es dann → DNSSEC-Problem

## Typische RZ-Szenarien

- **Migration**: TTL Tage vorher senken (z. B. 300 s), umstellen, mit `dig @resolver1 / @resolver2` prüfen, TTL wieder hoch
- **Mail kommt nicht an**: MX, A des MX, **PTR** des Mailservers (Reverse muss zum Hostnamen passen), SPF/DKIM/DMARC per TXT
- **AD-Anmeldung schlägt fehl**: SRV-Records der DCs, Client nutzt falschen DNS (z. B. 8.8.8.8 statt DC)
- **Zertifikat ungültig**: Name in DNS ≠ Name im Zertifikat (`openssl s_client -servername`)
- **Kubernetes**: `kubectl exec -it pod -- nslookup service.namespace.svc.cluster.local`

## Übungen

- [ ] `dig +trace` für die Domain deines Ausbildungsbetriebs – Ablauf erklären
- [ ] MX, SPF, DMARC einer großen Domain auslesen und interpretieren
- [ ] Eigene BIND-Zone im Labor, Serial absichtlich nicht erhöhen → Secondary bekommt Änderung nicht → mit `dig SOA @primary` / `@secondary` nachweisen
- [ ] TTL beobachten: zweimal hintereinander `dig` gegen einen Caching-Resolver
- [ ] Eintrag in `/etc/hosts` setzen und den Unterschied `dig` vs. `getent hosts` zeigen
