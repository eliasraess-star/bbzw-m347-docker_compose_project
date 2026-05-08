# Fragen – Integration der Services (DL10)

Name: <Nachname> <Vorname>
Klasse: <Klasse>

---

## 1. Services verstehen

Welche Services haben Sie in Ihrer `compose.yaml` definiert?

Antwort:
- api (FastAPI Backend)
- db (PostgreSQL Datenbank)
- adminer (Datenbank Web-Admin)
- frontend (Nginx mit HTML/CSS)

---

Welche Aufgabe hat jeder Service in Ihrem System?

Antwort:
- **api**: Bereitstellung der REST-API für Business-Logik und Datenverwaltung (Port 8000)
- **db**: Persistente Speicherung von Ticketboard-Daten mit PostgreSQL (Port 5432)
- **adminer**: Web-basiertes Admin-Tool zur Datenbankverwaltung (Port 8080)
- **frontend**: Benutzeroberfläche als statische HTML-Seite via Nginx (Port 3000)

---

## 2. Service-Kommunikation

Welchen Servicenamen verwendet die API, um die Datenbank zu erreichen?

Antwort:
Der Servicename ist `db`. Dies wird in der Umgebungsvariable `POSTGRES_HOST: db` konfiguriert. Der Verbindungsstring lautet: `postgresql://ticketuser:secret@db:5432/ticketdb`

---

Warum funktioniert `localhost` innerhalb eines Containers nicht für die Kommunikation mit anderen Services?

Antwort:
`localhost` (127.0.0.1) bezieht sich auf den eigenen Container. Jeder Container hat sein eigenes Netzwerk-Namespace. Die API im api-Container kann die Datenbank im db-Container nicht über `localhost` erreichen. Man benötigt den tatsächlichen Hostnamen oder die IP-Adresse des db-Containers.

---

Wie stellt Docker Compose sicher, dass sich Services gegenseitig finden können?

Antwort:
Docker Compose erstellt ein internes Netzwerk (default: `ticketboard_default`) und einen DNS-Service. Der Docker-Daemon übersetzt Servicenamen automatisch in IP-Adressen. Wenn die API sich mit `db` verbindet, löst Docker DNS den Namen zur aktuellen IP-Adresse des db-Containers auf.

---

## 3. Ports und Zugriff

Über welche Ports sind folgende Services erreichbar?

* API: Port 8000
* Adminer: Port 8080
* Frontend: Port 3000

---

Welcher Unterschied besteht zwischen:

* Container-Port: Der Port, auf dem der Service **innerhalb des Containers** läuft (z.B. Nginx läuft auf Port 80 im Container)
* Host-Port: Der Port, auf dem der Service vom **Host-System** aus erreichbar ist (z.B. Frontend über Port 3000 vom Host)

Antwort: Container-Port ist intern, Host-Port ist extern erreichbar. Das Port-Mapping `3000:80` bedeutet: Host-Port 3000 → Container-Port 80

---

## 4. Persistenz

Was passiert mit den Daten, wenn ein Container ohne Volume gelöscht wird?

Antwort:
Die Daten gehen verloren! Der Container speichert alle Daten im schreibbaren Dateisystem des Containers. Wenn der Container gelöscht wird, wird auch das Dateisystem gelöscht. Ein Volume ist notwendig, um Daten dauerhaft zu speichern.

---

Wie haben Sie die Persistenz für die Datenbank umgesetzt?

Antwort:
Ich habe ein benanntes Volume `postgres_data` in der compose.yml definiert und dieses im db-Service eingebunden:
```yaml
volumes:
  postgres_data:

services:
  db:
    volumes:
      - postgres_data:/var/lib/postgresql/data
```
Das Volume speichert die PostgreSQL-Daten außerhalb des Containers.

---

Warum ist ein Volume für die Datenbank notwendig?

Antwort:
Die Datenbank ist eine zustandsvolle Anwendung (stateful). Sie speichert wichtige Geschäftsdaten. Ohne Volume würden alle Einträge verloren gehen, wenn der db-Container neu gestartet wird. Ein Volume stellt sicher, dass Daten persistent sind und über Container-Neustarts hinweg erhalten bleiben.

---

## 5. Compose-Konfiguration

Welche Elemente haben Sie in Ihrer `compose.yaml` definiert?

Antwort:
- `services`: Definition der 4 Services (api, db, adminer, frontend)
- `build`: Für Services mit eigenen Dockerfiles (api, frontend)
- `image`: Für vorgefertigte Images (postgres, adminer)
- `ports`: Port-Mapping zwischen Host und Container
- `depends_on`: Abhängigkeiten zwischen Services
- `volumes`: Persistente Speicherung für PostgreSQL-Daten
- `environment` / `env_file`: Umgebungsvariablen für Konfiguration

---

Welche Umgebungsvariablen sind für die Datenbank-Verbindung notwendig?

Antwort:
```
POSTGRES_DB=ticketdb          # Name der Datenbank
POSTGRES_USER=ticketuser      # Benutzer für DB
POSTGRES_PASSWORD=secret      # Passwort für DB
POSTGRES_HOST=db              # Servicename der DB
POSTGRES_PORT=5432            # Port der DB
```
Diese werden aus der `.env`-Datei gelesen und in compose.yml verwendet.

---

Wofür wird `depends_on` verwendet?

Antwort:
`depends_on` definiert die Startabhängigkeiten zwischen Services. Z.B.:
```yaml
api:
  depends_on:
    - db
```
bedeutet: Der api-Container startet erst, nachdem der db-Container gestartet wurde. Dies garantiert nicht, dass die Datenbank ready ist, aber zumindest den Startorden.

---

## 6. Systemtest

Hat das System beim ersten Start vollständig funktioniert?

Antwort:
Nein, anfangs gab es einen Build-Fehler beim Frontend. Das System brauchte eine Iteration zur Behebung des Build-Kontextes.

---

Welche Probleme sind aufgetreten?

Antwort:
1. **Build-Fehler**: `COPY index.html` fehlgeschlagen
   - Grund: Falscher Build-Kontext (context: . statt context: ./frontend)
   - Fehler: `/index.html": not found`

2. **Image-Caching**: Alte Container-Images wurden wiederverwendet
   - Grund: Docker Compose nutzte gecachte Images
   - Auswirkung: Alte Konfiguration wurde geladen

---

Wie haben Sie diese Probleme gelöst?

Antwort:
1. **Build-Kontext korrigiert**: In compose.yml geändert:
   - ❌ `context: .` + `dockerfile: frontend/Dockerfile`
   - ✅ `context: ./frontend` + `dockerfile: Dockerfile`

2. **Sauberer Rebuild**:
   ```bash
   docker compose down
   docker image rm ticketboard-api ticketboard-frontend
   docker compose up -d --build
   ```

---

## 7. Verständnis

Beschreiben Sie kurz den Datenfluss in Ihrem System.

(Beispiel: Frontend → API → Datenbank)

Antwort:
```
Browser (Port 3000)
    ↓
Frontend (Nginx, Port 3000)
    ↓ (HTTP-Request zu /api/...)
API (FastAPI, Port 8000)
    ↓ (SQL-Queries)
Datenbank (PostgreSQL, Port 5432)
    ↓ (Persistiert Daten in Volume postgres_data)
Volume (postgres_data)
```

Zusätzlich: Adminer (Port 8080) kann direkt auf die Datenbank zugreifen für Admin-Operationen.

---

Was passiert beim Befehl:

```bash
docker compose down
```

Antwort:
1. Alle laufenden Container werden **gestoppt**
2. Alle Container werden **gelöscht**
3. Das Netzwerk `ticketboard_default` wird **gelöscht**
4. **NICHT gelöscht**: Images, Volumes (=Daten bleiben!)
5. Beim nächsten `docker compose up` werden neue Container aus den Images erstellt und das Volume wird wiederverwendet (→ Daten sind erhalten!)

---

## 8. Reflexion

Was war für Sie heute die wichtigste Erkenntnis?

Antwort:
Die wichtigste Erkenntnis ist, dass Docker Compose die Komplexität von Multi-Container-Systemen massiv vereinfacht. Statt manuelle Container zu verwalten und IP-Adressen zu tracken, genügt eine einfache YAML-Datei. Die automatische DNS-Auflösung (Servicenamen statt localhost) ist ein starkes Konzept, das Service-Discovery elegant löst.

Zusätzlich wurde mir bewusst, wie wichtig Volumes für zustandsvolle Anwendungen sind. Ohne Persistierung wäre das System bei jedem Neustart wertlos.

Auch das Debugging wurde zur wichtigen Fähigkeit: Build-Fehler lesen, Logs prüfen, systematisch vorgehen.

---

**Fazit**: Das System ist vollständig funktionsfähig und bereit für die nächste Entwicklungsphase (PW03).

Was war schwierig oder noch unklar?

Antwort:
