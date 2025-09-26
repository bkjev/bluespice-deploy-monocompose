## BlueSpice deploy – Single-file Docker Compose (All-in-one)

Dieser Fork konsolidiert die ursprüngliche, modular aufgebaute BlueSpice-Deployment-Struktur (mehrere Compose-Dateien und ein Wrapper-Skript) in eine einzige, übersichtliche docker-compose.yml.

### Was wurde geändert (High level)
- Alle relevanten Services aus `compose/docker-compose.*.yml` wurden in eine einzige `docker-compose.yml` zusammengeführt.
- Optionale Komponenten (z. B. Let’s Encrypt) sind weiterhin enthalten.
- Die internen Abhängigkeiten (prepare/upgrade, search) wurden in einem File konsistent abgebildet.
- Der interne DB-Service entfällt. Externe Datenbank wird über `.env` konfiguriert.

### Warum diese Änderung
- Weniger „Magie“ durch Wrapper-Skripte: Änderungen sind transparenter und dokumentiert.
- Deployable auf unserer Managed Docker-Infrastruktur

---

## Schnellstart

1) Umgebungsvariablen setzen

2) Start (ohne Let’s Encrypt):
```bash
docker compose up -d
```

3) Optional: Let’s Encrypt aktivieren (wenn konfiguriert):
```bash
docker compose up -d letsencrypt
```

Hinweis: Wenn kein interner Proxy genutzt wird, `wiki-web`-Port veröffentlichen und TLS extern terminieren (siehe Abschnitt Proxy/TLS unten).

---

## Services im Überblick

- `prepare`/`upgrade`: Einmalige Vor-/Upgrade-Schritte. `wiki-web` und `wiki-task` warten auf durchgelaufene Upgrades.
- `wiki-web`: BlueSpice Web-Frontend (Port 9090 intern). Persistenz unter `${DATADIR}/wiki`.
- `wiki-task`: Hintergrundaufgaben und Backups (`BACKUP_HOUR`). Nutzt dieselben Daten wie `wiki-web`.
- `search`: OpenSearch für die Volltextsuche. Persistenz unter `${DATADIR}/search`. `wiki-web`/`wiki-task` starten erst, wenn Suche bereit ist.
- Stateless-Services: `cache`, `pdf`, `formula`, `diagram` (optional via Proxy-Routing erreichbar).
- `proxy`: NGINX-Proxy mit automatisiertem Routing via `VIRTUAL_*`-Variablen.
- `letsencrypt`: ACME-Companion für automatische Zertifikatserneuerung, benötigt den `proxy`.

Optionale Komponenten wie `collabpads`/`antivirus` können bei Bedarf analog ergänzt werden (im Originalprojekt enthalten; hier ist der Kern konsolidiert).

---

## Proxy/TLS-Strategien

Es gibt zwei gängige Wege für HTTPS:

1) Interner Proxy mit Let’s Encrypt
- Starte `proxy` und optional `letsencrypt`.
- `wiki-web` setzt `VIRTUAL_HOST`, `VIRTUAL_PATH`, `VIRTUAL_PORT` (und bei LE `LETSENCRYPT_HOST`).
- `docker compose up -d` (und optional `letsencrypt`).

2) Externer TLS-Terminierer (Reverse Proxy / Load Balancer)
- `proxy` und `letsencrypt` weglassen.
- In `wiki-web` einen Port veröffentlichen (z. B. `"9090:9090"`).
- TLS extern terminieren und per HTTP an `wiki-web:9090` weiterleiten.
- In `.env` `WIKI_PROTOCOL=https`, `WIKI_HOST=<FQDN>`, `WIKI_PORT=443` passend setzen.

Tipp: Entscheide dich für genau eine Strategie, um doppelte TLS-Terminierung zu vermeiden.

---

## Externe Datenbank nutzen

Wenn eine externe MySQL/MariaDB verwendet wird, setze diese Variablen in `.env`:
- `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`
- Optional (für Upgrade-Szenarien): `DB_ROOT_USER`, `DB_ROOT_PASS`

Entferne oder starte keinen internen `database`-Service. Stelle Netzwerkzugriff aus den Containern zur externen DB sicher (Firewall/DNS). Unter Docker Desktop kann `host.docker.internal` als Hostname helfen, wenn die DB auf demselben Host läuft.

Suche bleibt lokal: `search`-Service läuft weiter als Container.

---

## Wichtige Umgebungsvariablen

Siehe auch `README_ORIGINAL.md` für eine detaillierte Tabelle. Mindestens:
- `DATADIR` – Pfad für persistente Datenvolumes
- `VERSION`/`EDITION` bzw. `BLUESPICE_WIKI_IMAGE` – Imageauswahl
- `WIKI_HOST`, `WIKI_PORT`, `WIKI_PROTOCOL`, `WIKI_BASE_PATH`
- DB-Variablen wie oben
- Optional Proxy/TLS: `ADMIN_MAIL` (Let’s Encrypt)

---

## Änderungen am Upstream (Original-Repo) übernehmen

Das Original-Repo (siehe `README_ORIGINAL.md`) wird weiterentwickelt. Die Inhalte von `compose/` sind unverändert und könne überschrieben werden. Anpassungen sollte aber in unsere `docker-compose.yml`übertragen werden.

---

## Referenzen
- Upstream-Doku: `https://en.wiki.bluespice.com/wiki/Setup:Installation_Guide/Docker`
- Original-Repo README: siehe `README_ORIGINAL.md`


