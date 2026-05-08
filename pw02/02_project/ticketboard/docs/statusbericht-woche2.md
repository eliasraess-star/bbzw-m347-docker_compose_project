# Statusbericht – Woche 2: Integration der Services

**Datum**: Mai 2026  
**Aufgabe**: Integration der Docker Compose Services  
**Status**: ✅ Abgeschlossen

---

## Umgesetzte Arbeiten

### 1. Compose-Konfiguration erstellt

- **Datei**: `compose.yml`
- **Services definiert**: 4 (api, db, adminer, frontend)
- **Konfigurationselemente**:
  - Ports für externen Zugriff
  - Umgebungsvariablen aus `.env`
  - Abhängigkeiten mit `depends_on`
  - Volumes für Persistenz

### 2. Alle Services integriert

| Service | Status | Port | Besonderheiten |
|---------|--------|------|-----------------|
| API | ✅ Läuft | 8000 | FastAPI auf Python 3.12 |
| Datenbank | ✅ Läuft | 5432 | PostgreSQL 16-Alpine |
| Adminer | ✅ Läuft | 8080 | Web-Admin für PostgreSQL |
| Frontend | ✅ Läuft | 3000 | Nginx mit HTML-UI |

### 3. Service-Kommunikation hergestellt

- API verbindet sich via Servicenamen zu `db`
- Frontend kommuniziert mit API
- Adminer greift auf Datenbank zu
- DNS-Auflösung funktioniert automatisch

### 4. Persistenz implementiert

- Volume `postgres_data` erstellt
- Eingebunden bei: `/var/lib/postgresql/data`
- Daten bleiben nach `docker compose down` erhalten

### 5. Dokumentation erstellt

- ✅ `docs/architektur.md` – Detaillierte Systemübersicht
- ✅ `docs/questions.md` – Fragen beantwortet
- ✅ `docs/statusbericht-woche2.md` – Dieser Bericht

---

## Aktueller Stand

### System-Status
```bash
$ docker compose ps
NAME                     IMAGE                  STATUS
ticketboard-adminer-1    adminer                Up 5 minutes
ticketboard-api-1        ticketboard-api        Up 5 minutes
ticketboard-db-1         postgres:16-alpine     Up 5 minutes
ticketboard-frontend-1   ticketboard-frontend   Up 5 minutes
```

### Funktionalitätstest
- ✅ API erreichbar: `curl http://localhost:8000/health` → `{"status": "ok"}`
- ✅ Frontend erreichbar: `http://localhost:3000` → HTML lädt
- ✅ Adminer erreichbar: `http://localhost:8080` → Web-UI sichtbar
- ✅ Datenbank erreichbar: Port 5432 offen
- ✅ Persistenz aktiv: Volume vorhanden

---

## Aufgetretene Probleme

### Problem 1: Build-Kontext für Frontend
**Beschreibung**: Dockerfile konnte `index.html` nicht finden
```
ERROR: failed to calculate checksum of ref: "/index.html": not found
```

**Ursache**: Falscher Build-Kontext in compose.yml
- ❌ Falsch: `context: .` mit `dockerfile: frontend/Dockerfile`
- ✅ Richtig: `context: ./frontend` mit `dockerfile: Dockerfile`

**Lösung**: Build-Kontext auf `./frontend` gesetzt

### Problem 2: Alte Image-Versionen
**Beschreibung**: Alte API-Container liefen mit veralteter main.py

**Ursache**: Docker-Image-Cache mit alte Konfiguration

**Lösung**: 
```bash
docker compose down
docker image rm ticketboard-api ticketboard-frontend
docker compose up -d --build
```

---

## Nächste Schritte (PW03)

1. **Datenbankverbindung von API ausbauen**
   - Tabellen erstellen
   - Datenbankoperationen implementieren

2. **Frontend erweitern**
   - API-Calls implementieren
   - Formular für Ticketeingabe

3. **Error-Handling verbessern**
   - Health-Checks verfeinern
   - Fehlerbehandlung in API

4. **Monitoring/Logging**
   - Docker-Logs strukturieren
   - Fehlerbehandlung erweitern

---

## Lessons Learned

### Was funktioniert gut:
✅ Docker Compose vereinfacht Multi-Container-Orchestrierung  
✅ Service-Namen (DNS) funktionieren zuverlässig  
✅ Volumes stellen Persistenz sicher  
✅ Abhängigkeiten mit `depends_on` sind hilfreich  

### Was war knifflig:
⚠️ Build-Kontexte müssen exakt passen  
⚠️ Image-Caching kann zu unerwarteten Problemen führen  
⚠️ `depends_on` garantiert nicht, dass Service ready ist  

### Erkenntnisse:
💡 Servicenamen statt localhost = automatische Service-Discovery  
💡 Volumes sind essentiell für Datenpersistenz  
💡 Systemarchitektur wird durch compose.yml sichtbar  
💡 Testen ist wichtig: Ports, Erreichbarkeit, Logs prüfen  

---

## Abgabe-Checkliste

- [x] compose.yml vollständig konfiguriert
- [x] Alle 4 Services starten und sind erreichbar
- [x] Datenbank persistiert Daten
- [x] API erreichbar unter `http://localhost:8000/health`
- [x] Frontend erreichbar unter `http://localhost:3000`
- [x] Adminer erreichbar unter `http://localhost:8080`
- [x] docs/architektur.md erstellt
- [x] docs/statusbericht-woche2.md erstellt (dieser Report)
- [x] docs/questions.md beantwortet

---

**Fazit**: Das Mehrcontainer-System ist vollständig funktionsfähig und kann als Basis für weitere Entwicklung in PW03 verwendet werden.
