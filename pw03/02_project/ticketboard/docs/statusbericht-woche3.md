# Statusbericht – Woche 3 (DL11)

Name: <Nachname> <Vorname>
Klasse: <Klasse>

---

## Umgesetzte Arbeiten

- Hardcodierte Werte in `compose.yml` und `app/main.py` analysiert und als Konfigurationswerte identifiziert (DB-Name, User, Passwort, Host, Port).
- `.env`-Datei im Projektordner angelegt mit allen nötigen Variablen (`POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_HOST`, `POSTGRES_PORT`).
- `.gitignore` erstellt (`.env`, `__pycache__/`, `*.pyc`, `.venv/`), damit Secrets nicht im Repo landen.
- `.env.example` geprüft: enthält alle Schlüssel aus `.env`, aber keine echten Passwörter.
- `compose.yml` auf Variablensubstitution umgestellt – sowohl im `api`- als auch im `db`-Service. `DATABASE_URL` wird jetzt aus den Einzelvariablen zusammengesetzt.
- `app/main.py` angepasst: `DATABASE_URL` wird via `os.getenv("DATABASE_URL")` gelesen statt hardcodiert.
- Auflösung mit `docker compose config` überprüft – alle `${VARIABLE}` werden korrekt ersetzt.
- System mit `docker compose up --build` gestartet und alle Endpunkte getestet.
- Neustart-Test (`docker compose down` → `up --build`) erfolgreich durchgeführt.
- `docs/questions.md` beantwortet.

---

## Aktueller Stand

- System startet sauber mit einem Befehl.
- `/health` liefert `{"status": "ok"}`.
- `/db-check` liefert `{"db": "connected"}` – Datenbankverbindung steht.
- Adminer (Port 8080) und Frontend (Port 3000) sind erreichbar.
- Keine Passwörter mehr im Code oder in `compose.yml`.
- `.env` lokal vorhanden, aber durch `.gitignore` vom Repo ausgeschlossen.

---

## Offene Probleme / Herausforderungen

- 

---

## Nächste Schritte

- Nächsten Auftrag lösen

---

## Selbsteinschätzung (optional)

Der Auftrag hat geholfen, den Unterschied zwischen Code und Konfiguration sauber zu trennen. Besonders der Mechanismus mit `${VARIABLE}` in `compose.yml` und das Zusammenspiel von `.env`, `.gitignore` und `.env.example` ist jetzt klar. `docker compose config` werde ich ab jetzt häufiger zum Debuggen nutzen.