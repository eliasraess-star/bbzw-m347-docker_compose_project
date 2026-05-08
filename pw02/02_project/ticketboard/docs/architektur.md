# TicketBoard – Systemarchitektur

## Übersicht

Das TicketBoard ist ein Mehrcontainer-System, das mit Docker Compose orchestriert wird. Es besteht aus vier Hauptkomponenten, die über ein gemeinsames Netzwerk miteinander kommunizieren.

## Systemkomponenten

### 1. Frontend (Web-UI)
- **Image**: `ticketboard-frontend` (Nginx)
- **Port (Host)**: 3000
- **Port (Container)**: 80
- **Aufgabe**: Präsentation der Benutzeroberfläche
- **Kommunikation**: HTTP zu API (Port 8000)

### 2. API (Backend)
- **Image**: `ticketboard-api` (Python/FastAPI)
- **Port (Host)**: 8000
- **Port (Container)**: 8000
- **Aufgabe**: Business-Logik und Datenverwaltung
- **Kommunikation**: PostgreSQL-Verbindung zur Datenbank (Service: `db`)

### 3. Datenbank (PostgreSQL)
- **Image**: `postgres:16-alpine`
- **Port (Host)**: 5432
- **Port (Container)**: 5432
- **Aufgabe**: Persistente Speicherung von Daten
- **Persistenz**: Volume `postgres_data` → `/var/lib/postgresql/data`

### 4. Adminer (DB-Admin-Tool)
- **Image**: `adminer`
- **Port (Host)**: 8080
- **Port (Container)**: 8080
- **Aufgabe**: Web-basierte Verwaltung der PostgreSQL-Datenbank
- **Kommunikation**: Direkter Zugriff auf Datenbank (Service: `db`)

## Kommunikationsfluss

```
Browser (localhost)
    ↓
    ├─→ Frontend (Port 3000)
    │       ↓
    │   API (Port 8000)
    │       ↓
    │   PostgreSQL (Port 5432)
    │       ↓
    │   Adminer (Port 8080)
    │       ↓
    │   PostgreSQL (Port 5432)
    └─→ Datenbank → Volume (postgres_data)
```

## Service-Verbindungen

### DNS-Auflösung in Docker Compose

Docker Compose erstellt automatisch einen internen DNS-Service. Services können sich gegenseitig über ihre **Servicenamen** erreichen:

| Service | DNS-Name | Port |
|---------|----------|------|
| Frontend | `frontend` | 80 |
| API | `api` | 8000 |
| PostgreSQL | `db` | 5432 |
| Adminer | `adminer` | 8080 |

**Beispiel**: Die API verbindet sich zur Datenbank via:
```
postgresql://ticketuser:secret@db:5432/ticketdb
```

Der Hostname `db` wird automatisch zum PostgreSQL-Container aufgelöst.

## Abhängigkeiten (depends_on)

```yaml
api:
  depends_on:
    - db           # API wartet auf Datenbank

adminer:
  depends_on:
    - db           # Adminer braucht DB

frontend:
  depends_on:
    - api          # Frontend braucht API
```

Diese Reihenfolge garantiert den korrekten Start der Services, ist aber kein Garant für Readiness (z.B. kann die DB noch nicht vollständig initialisiert sein).

## Persistenz

### Volume-Definition

```yaml
volumes:
  postgres_data:    # Benanntes Volume
```

### Verwendung im db-Service

```yaml
db:
  volumes:
    - postgres_data:/var/lib/postgresql/data
```

- **Host-Seite**: Benanntes Volume `postgres_data` (Docker verwaltet Speicherort)
- **Container-Seite**: `/var/lib/postgresql/data` (PostgreSQL Datenverzeichnis)

### Persistenz-Garantie

Nach `docker compose down`:
- Container werden gelöscht ✓
- Images bleiben erhalten ✓
- Volume bleibt erhalten ✓ (mit Daten!)

Nach `docker compose up`:
- Neue Container werden erstellt
- Altes Volume wird erneut eingebunden
- **Daten sind erhalten!**

## Umgebungsvariablen

Die Datenbank wird über `.env` konfiguriert:

```
POSTGRES_DB=ticketdb
POSTGRES_USER=ticketuser
POSTGRES_PASSWORD=secret
POSTGRES_HOST=db
POSTGRES_PORT=5432
```

Diese werden im Service mit `env_file` geladen und in `environment` weitergeleitet.

## Netzwerk

Docker Compose erstellt automatisch ein Netzwerk `ticketboard_default` mit folgende Eigenschaften:

- Alle Services sind im gleichen Netzwerk
- Services können sich über Servicenamen erreichen (DNS)
- Externe Zugriffe nur über konfigurierte Ports möglich

## Start und Lifecycle

```bash
# System starten
docker compose up -d --build

# Logs anschauen
docker compose logs -f api

# Status überprüfen
docker compose ps

# System herunterfahren (Container weg, Daten bleiben)
docker compose down

# Vollständiger Reset (auch Daten)
docker compose down -v
```

## Zusammenfassung

Das TicketBoard-System demonstriert:
1. **Multi-Container-Orchestrierung** mit Docker Compose
2. **Service-Discovery** durch DNS-Auflösung (Servicenamen statt IP)
3. **Persistenz** durch Volumes
4. **Abhängigkeitsverwaltung** mit `depends_on`
5. **Port-Mapping** zwischen Host und Containern
6. **Umgebungsvariablen** für flexible Konfiguration
