# INCIDENT REPORT
## Paperclip: Issue-Erstellung schlägt fehl (HTTP 500)

**Incident-ID:** INC-2026-04-06-001
**Datum:** 06. April 2026
**Schweregrad:** MEDIUM (Betrieb eingeschränkt, keine Datenverlust)
**Status:** RESOLVED
**Bearbeiter:** Thomas Z.
**Dauer:** ca. 2 Stunden

---

## 1. Zusammenfassung

Die Erstellung von Issues in Paperclip über die Web-UI und die REST-API schlug mit HTTP 500 "Internal server error" fehl. Ursache war ein desynchronisierter Issue-Identifier-Counter in der Datenbank: Paperclip versuchte den Identifier `SYN-33` zu vergeben, dieser existierte jedoch bereits in der `issues`-Tabelle, was einen PostgreSQL Unique-Constraint-Fehler auslöste.

---

## 2. Symptome

| Symptom | Detail |
|---|---|
| Web-UI zeigt "Internal server error" | Beim Klick auf "Create Issue" im Paperclip-Frontend |
| API gibt HTTP 500 zurück | `POST /api/companies/{id}/issues` → `{"error":"Internal server error"}` |
| Kein erkennbarer Fehler in Access-Logs | Nur `500` im HTTP-Log, kein Stack Trace |
| Betrifft alle Projekte | Nicht projekt-spezifisch |

---

## 3. Ursache (Root Cause)

### Technisch
PostgreSQL Unique-Constraint-Verletzung auf `issues.identifier`:

```
PostgresError {
  code: "23505",
  detail: "Key (identifier)=(SYN-33) already exists.",
  constraint_name: "issues_identifier_idx",
  table_name: "issues"
}
```

### Hintergrund
Paperclip speichert einen globalen Issue-Zähler in der Tabelle `companies` als Feld `issue_counter`. Bei der Erstellung eines neuen Issues inkrementiert Paperclip diesen Zähler und generiert den Identifier (z.B. `SYN-33`).

In einem früheren Nordstern-Reset wurden Issues **direkt in der Datenbank** via `TRUNCATE TABLE issues CASCADE` gelöscht — ohne den `issue_counter` zurückzusetzen. Dadurch war der Counter auf `32`, die Issues-Tabelle aber leer. Paperclip versuchte beim nächsten `POST` den Identifier `SYN-33` zu vergeben, der jedoch laut Counter bereits "vergeben" sein sollte — und scheiterte am Unique-Index, weil ein anderer Code-Pfad die Vergabe anders handhabte.

---

## 4. Diagnose-Pfad

### Schritt 1 — Fehlerquelle eingrenzen
Web-UI zeigte denselben Fehler wie die API → Problem liegt im Backend, nicht im Client.

### Schritt 2 — Logs finden
Paperclip läuft **nicht in Docker**, sondern als nativer Node.js-Prozess:
```bash
ps aux | grep -E 'node|paperclip'
# → sudo -u paperclip nohup paperclipai run
```
Log-Pfad: `/home/paperclip/paperclip.log` (nicht `/home/paperclipai/`)

### Schritt 3 — Echten Fehler aus Log lesen
```bash
# Auf Proxmox Host:
pct exec 110 -- bash -c "
tail -f /home/paperclip/paperclip.log &
sleep 1
curl -X POST -H 'Authorization: Bearer {TOKEN}' \
  -H 'Content-Type: application/json' \
  -d '{\"title\":\"Test\",\"status\":\"todo\"}' \
  'http://localhost:3100/api/companies/{COMPANY_ID}/issues'
sleep 3; kill %1"
```
→ Stack Trace zeigt PostgresError code 23505

### Schritt 4 — DB-Zugang
Embedded PostgreSQL auf Port 54329, Auth-Methode: `password` (in `pg_hba.conf`).
Für direkten Zugriff: `pg_hba.conf` temporär auf `trust` setzen + SIGHUP:
```bash
# pg_hba.conf Pfad:
/home/paperclip/.paperclip/instances/default/db/pg_hba.conf

# PostgreSQL PID und Reload:
PG_PID=$(pgrep -f 'postgres.*54329' | head -1)
kill -HUP $PG_PID
```

### Schritt 5 — Counter und Issues prüfen
```sql
SELECT identifier, title FROM issues ORDER BY identifier DESC LIMIT 20;
SELECT id, name, issue_counter FROM companies;
```
→ `issue_counter = 32`, aber Issues-Tabelle enthielt verwaiste Einträge

---

## 5. Lösung

### Sofortmaßnahme (angewendet)

```bash
# 1. pg_hba.conf auf trust setzen
cat > /home/paperclip/.paperclip/instances/default/db/pg_hba.conf << 'EOF'
local   all   all                trust
host    all   all   127.0.0.1/32 trust
host    all   all   ::1/128      trust
EOF

# 2. PostgreSQL reload
PG_PID=$(pgrep -f 'postgres.*54329' | head -1)
kill -HUP $PG_PID && sleep 2

# 3. Issues + abhängige Tabellen leeren (CASCADE wegen FKs)
psql -h 127.0.0.1 -p 54329 -U paperclip -d paperclip \
  -c "TRUNCATE TABLE issues CASCADE;"

# 4. Counter zurücksetzen
psql -h 127.0.0.1 -p 54329 -U paperclip -d paperclip \
  -c "UPDATE companies SET issue_counter = 0;"

# 5. pg_hba.conf wiederherstellen
cp /home/paperclip/.paperclip/instances/default/db/pg_hba.conf.bak \
   /home/paperclip/.paperclip/instances/default/db/pg_hba.conf
kill -HUP $PG_PID
```

### Verifizierung
```sql
SELECT issue_counter FROM companies;
-- Erwartet: 0

SELECT COUNT(*) FROM issues;
-- Erwartet: 0
```
Danach: Issue-Erstellung über Web-UI und API erfolgreich getestet.

---

## 6. Weitere Stolpersteine (Lessons Learned)

### A — Falscher Port
Jarvis (Agent auf .96) hat initial Port 80 (InvoiceNinja) statt Port 3100 (Paperclip) angesprochen. Beide laufen auf LXC 110.

**Merke:**
| Service | LXC | Port |
|---|---|---|
| InvoiceNinja | 110 | 80 / 443 |
| Paperclip | 110 | 3100 |

### B — "Host not allowed"
Paperclip erlaubt API-Requests nur von `allowedHostnames` (config.json). Externe Hosts bekommen `"Host not allowed"` statt einem HTTP-Fehlercode. Requests müssen von `.96` (openclaw) oder `localhost` kommen.

### C — Falscher Modell-String
Beim Versuch Jarvis auf Sonnet umzustellen wurde ein ungültiger Modell-String eingetragen (`claude-sonnet-4-6-20250514` statt `claude-haiku-4-5-20251001`). Jarvis war danach nicht mehr erreichbar.

**Fix:**
```bash
ssh claw@192.168.178.96
sed -i 's/anthropic\/claude-sonnet-4-6-20250514/anthropic\/claude-haiku-4-5-20251001/g' \
  /home/claw/.openclaw/openclaw.json
pkill -f "openclaw run" && sleep 2
nohup openclaw run > /tmp/openclaw.log 2>&1 &
```

### D — TRUNCATE CASCADE löscht mehr als erwartet
`TRUNCATE TABLE issues CASCADE` leert auch:
- `issue_comments`
- `issue_read_states`
- `issue_approvals`
- `issue_attachments`
- `issue_labels`
- `cost_events`
- `workspace_runtime_services`

Das ist bei einem Nordstern-Reset gewünscht, muss aber dokumentiert sein.

---

## 7. Präventivmaßnahmen

### P1 — Issues NIE direkt in der DB löschen
Issues immer über die Paperclip API löschen:
```
DELETE /api/companies/{companyId}/issues/{issueId}
```
Die API dekrementiert `issue_counter` korrekt. Direkter DB-Eingriff lässt den Counter stehen.

### P2 — Bei Nordstern-Reset: Counter explizit zurücksetzen
Wenn ein DB-Eingriff unvermeidlich ist:
```sql
TRUNCATE TABLE issues CASCADE;
UPDATE companies SET issue_counter = 0;
```
Beide Schritte immer zusammen ausführen.

### P3 — Modell-String vor Änderung verifizieren
Gültige Modell-Strings für openclaw.json (Stand April 2026):
```
anthropic/claude-haiku-4-5-20251001   ← Standard Jarvis
anthropic/claude-sonnet-4-5-20251022  ← Sonnet (falls verfügbar)
```
Nach Änderung immer testen: kurze Nachricht an Jarvis senden und auf Antwort warten bevor weitere Arbeit delegiert wird.

### P4 — pg_hba.conf Backup immer anlegen
Vor jedem pg_hba.conf Edit:
```bash
cp /home/paperclip/.paperclip/instances/default/db/pg_hba.conf \
   /home/paperclip/.paperclip/instances/default/db/pg_hba.conf.bak
```
Nach dem DB-Eingriff sofort wiederherstellen.

### P5 — Jarvis SSH-Keys auf alle LXCs
Jarvis (.96) braucht SSH-Zugang auf LXC 110 für Debugging.
Einrichten (Thomas führt einmalig aus):
```bash
# SSH Key von openclaw auf LXC 110 autorisieren
ssh root@192.168.178.110 \
  "mkdir -p /root/.ssh && \
   echo '$(ssh claw@192.168.178.96 cat /home/claw/.ssh/id_rsa.pub)' \
   >> /root/.ssh/authorized_keys"
```

---

## 8. Infrastruktur-Referenz

| Komponente | Wert |
|---|---|
| Paperclip Host | LXC 110 / 192.168.178.110 |
| Paperclip Port | 3100 |
| Paperclip User | paperclip |
| Paperclip Log | `/home/paperclip/paperclip.log` |
| PostgreSQL Port | 54329 (embedded) |
| pg_hba.conf | `/home/paperclip/.paperclip/instances/default/db/pg_hba.conf` |
| DB Name | paperclip |
| DB User | paperclip |
| Company ID | `2cb965f3-f259-4955-9efe-3aaee47471c6` |
| OpenClaw Host | 192.168.178.96 |
| OpenClaw User | claw |
| OpenClaw Config | `/home/claw/.openclaw/openclaw.json` |

---

## 9. Verwandte Dokumente

- `SYNTIQAI-NORDSTERN-2026-04-06.md` — Architektur & Governance
- `JARVIS-VOLLPAKET-2026-04-06.md` — Vollständige Ausführungsanweisung
- Paperclip SKILL.md — API-Referenz

---

*Erstellt: 06. April 2026 | SyntiqAI | Thomas Z.*
