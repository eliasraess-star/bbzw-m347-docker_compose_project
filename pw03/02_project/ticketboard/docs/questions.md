# Fragen – Konfiguration und Umgebungsvariablen (DL11)

Name: <Elias> <Raess>
Klasse: <INA24dL>

---

## 1. Konfiguration

Welche Werte waren ursprünglich hardcoded in `compose.yml` und `app/main.py`?

Antwort: POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_HOST und POSTGRES_PORT waren in compose.yml hardcoded. In app/main.py wurde DATABASE_URL später durch os.getenv() aus Umgebungsvariablen gelesen.

---

Warum ist es ein Problem, Passwörter direkt in `compose.yml` einzutragen?

Antwort: Sicherheitsrisiko: Passwörter sind sichtbar in der Konfigurationsdatei. Git-Risiko: Wenn ins Repository committed, ist das Passwort im gesamten Git-Verlauf sichtbar. Jeder mit Repo-Zugriff könnte die Passwörter sehen. Produktion: Gleiches Passwort in Entwicklung und Produktion.

---

Was ist der Unterschied zwischen `.env` und `.env.example`?

Antwort: `.env` enthält echte Werte (mit echten Passwörtern), wird lokal verwendet und ist in .gitignore. `.env.example` ist eine Vorlage für andere Entwickler, zeigt welche Variablen nötig sind, hat Placeholder-Werte (z.B. "changeme") und wird ins Repository committed.

---

Warum muss `.env` in `.gitignore` eingetragen sein?

Antwort: Um zu verhindern, dass sensitive Daten ins Git-Repository gelangen. Schutz vor Datenlecks: Geheimnisse bleiben lokal sicher. Unterschiedliche Umgebungen: Jeder Entwickler kann eigene lokale Werte haben ohne Git-Konflikte.

---

## 2. Variablen in Compose

Wie referenziert man eine Variable aus `.env` in `compose.yml`?

Antwort: Mit der Syntax `${VARIABLE_NAME}`. Beispiel: `DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}`. Docker Compose liest automatisch `.env` und ersetzt die Variablen.

---

Was passiert, wenn eine Variable in `.env` fehlt, aber in `compose.yml` verwendet wird?

Antwort: Docker Compose gibt eine Warnung oder einen Fehler aus. Die Variable bleibt leer oder wird nicht ersetzt. Der Service kann nicht starten, wenn die fehlende Variable kritisch ist. Besser: Docker Compose sollte mit Fehlermeldung abbrechen.

---

Was zeigt der Befehl `docker compose config`? Wann ist er nützlich?

Antwort: Zeigt die endgültige/aufgelöste Konfiguration nach Variablen-Substitution. Alle ${VARIABLE} werden durch ihre Werte aus `.env` ersetzt. Nützlich für: Debugging (überprüfen ob Variablen korrekt ersetzt wurden), Validierung (prüfen ob Syntax korrekt ist), Dokumentation (sehen wie tatsächliche Konfiguration aussieht).

---

## 3. Dockerfile und Build

Warum wird `requirements.txt` in einem eigenen `COPY`-Schritt vor dem App-Code kopiert?

Antwort: Docker Layer Caching: Wenn nur App-Code sich ändert, muss `pip install` nicht erneut laufen. Performanz: Die Python-Packages sind bereits im Cache-Layer vorhanden. Weniger Build-Zeit: Nur der letzte COPY-Schritt (App-Code) wird erneut ausgeführt. Best Practice: Stabile Dateien zuerst kopieren.

---

Was bewirkt `.dockerignore`? Welche Dateien sollten darin stehen?

Antwort: `.dockerignore` teilt Docker mit, welche Dateien/Verzeichnisse beim COPY nicht ins Image eingebunden werden. Reduziert Image-Größe und Sicherheit. Sollten darin stehen: `.env` (Geheimnisse), `.git` (Git-History), `__pycache__/` (Python Cache), `*.pyc`, `.venv/` (Virtual Environment), `node_modules/`, `.dockerignore` selbst.

---

## 4. Systemtest

Funktioniert `/db-check` nach Ihrer Konfigurationsanpassung?

Antwort: Ja, wenn alle Umgebungsvariablen in `.env` gesetzt sind, der PostgreSQL-Container läuft und erreichbar ist, und die DATABASE_URL korrekt zusammengesetzt ist. Test: `curl http://localhost:8000/db-check`

---

Was zeigt der Endpunkt `/db-check` an, wenn die Verbindung funktioniert?

Antwort: Bei erfolgreicher Verbindung: `{"db": "connected"}`. Bei Fehler: `{"db": "error", "detail": "<Fehlermeldung>"}`. Der Endpunkt versucht eine SELECT 1 Query auszuführen. Damit wird überprüft, ob die Datenbankverbindung tatsächlich funktioniert.

---

## 5. Reflexion

Was war der wichtigste Schritt in dieser Woche?

Antwort: Verschieben von hardcoded Werten zu Umgebungsvariablen (.env). Verstehen warum Sicherheit/Secrets nicht in Konfigurationsdateien gehören. Der Unterschied zwischen `.env` (echte Werte) und `.env.example` (Vorlage). Praktisches Wissen wie Docker Compose Variablen einsetzt. Einsehen des wichtigsten DevOps/Security Prinzips: "Configuration vs. Secrets Management".

---

Was ist noch unklar oder möchten Sie besser verstehen?

Antwort: [Hier kann der Studierende persönliche offene Fragen oder Unsicherheiten dokumentieren]
