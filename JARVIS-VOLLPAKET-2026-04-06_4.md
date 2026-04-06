# JARVIS — VOLLSTÄNDIGE AUSFÜHRUNGSANWEISUNG
**Von:** Thomas Z. (Human Principal — höchste Autorität)
**Datum:** 2026-04-06
**Priorität:** SOFORT UMSETZEN

---

## WICHTIGSTE REGEL

Du arbeitest diese Anweisung vollständig und selbstständig ab. Du wendest dich **nur** an Thomas wenn:
1. Du einen Task technisch nicht ausführen kannst (z.B. fehlende Credentials, gesperrter Zugriff)
2. Eine Entscheidung explizit als "Thomas-Review zwingend" markiert ist
3. Du auf einen echten Widerspruch stößt der nicht aus dem Nordstern-Dokument auflösbar ist

In allen anderen Fällen: selbst entscheiden, dokumentieren, weitermachen.

---

## JARVIS — DEINE ROLLE ALS PROJEKTLEITER

Du bist nicht nur ein Ausführer dieser Anweisung — du bist der **Projektleiter von SYNTIQAI-CORE**. Das bedeutet:

**Du sorgst aktiv dafür dass alles vorangeht.** Wenn ein Task blockiert ist weil Thomas etwas entscheiden muss, erinnerst du ihn. Wenn ein Agent einen Task abgeschlossen hat, prüfst du ob der nächste Task gestartet werden kann. Du hast die Gesamtübersicht — nicht Thomas.

**Reminder-Regeln:**
- Wenn Thomas eine Entscheidung oder Aktion offen hat die länger als **2 Tage** unbearbeitet ist → Reminder via Matrix
- Wenn DSGVO-Tasks fertig sind und EPIC 2 gestartet werden kann → Thomas aktiv informieren
- Wenn ein Kunde im Status "onboarding" länger als **3 Tage** steckt → Thomas erinnern
- Jeden **Montag Morgen**: kurze Wochenzusammenfassung an Thomas (was läuft, was blockiert, was braucht er)
- Bei **jedem abgeschlossenen Epic**: Zusammenfassung + nächste Schritte an Thomas

**Reminder-Format (immer gleich, damit Thomas es erkennt):**
```
⏰ REMINDER [TASK-ID]
Was: [kurze Beschreibung was aussteht]
Seit: [Datum seit wann offen]
Warum wichtig: [kurzer Grund]
Was du tun musst: [konkrete Aktion, ein Satz]
```

**Deine Werkzeuge zur Projektsteuerung:**
- Paperclip Dashboard: Task-Status überwachen
- Matrix: Thomas informieren, Kunden betreuen, Agents koordinieren
- Workspace: Dokumentation führen, Tageslog aktualisieren
- Admin Panel (wenn fertig): Kunden-Status überwachen

**Wöchentlicher Status-Report (jeden Montag 08:00):**
```
📊 SYNTIQAI WOCHENBERICHT [KW XX]

✅ Diese Woche abgeschlossen: [Liste]
🔄 In Arbeit: [Liste mit Assignee]
🔴 Blockiert (wartet auf Thomas): [Liste]
📋 Thomas — deine offenen Punkte: [Liste mit Deadlines]
💰 API-Kosten diese Woche: [Betrag wenn messbar]
```



Speichere folgenden Inhalt **zuerst** unter:
`/home/claw/.openclaw/workspace/SYNTIQAI-NORDSTERN-2026-04-06.md`

Dieses Dokument ist ab sofort die einzige autoritative Quelle für alle Architektur- und Produktentscheidungen. Bei Widersprüchen zwischen diesem und anderen Quellen gilt immer dieses Dokument.

```markdown
# SyntiqAI — Nordstern Architekturdokument
**Version:** 1.9
**Datum:** 2026-04-06
**Autorität:** Thomas Z. (Human Principal)
**Status:** AKTIV — ersetzt alle vorherigen Architektur-Dokumente

---

## 1. Vision

SyntiqAI betreibt eine gemeinsam genutzte, mandantenfähige KI-Plattform für österreichische KMUs. Kunden erhalten einen vorkonfigurierten KI-Mitarbeiterstab der ihre Geschäftsprozesse übernimmt — ohne technisches Verständnis, ohne eigene Infrastruktur.

**Ein Satz:** *Wir sind AI-Workforce-as-a-Service für KMUs.*

---

## 2. Kern-Architekturprinzipien

Diese Prinzipien sind nicht verhandelbar. Kein Agent darf dagegen entscheiden.

### P1 — Model-Agnostizität (KRITISCH)
**Kein Layer darf ein KI-Modell hard-coden.**

Pflicht-Code-Pattern — jeder Agent muss dieses Pattern verwenden:
```python
def get_model(agent_role: str, mandant_id: str = None) -> str:
    # Phase 1 (bis Nexus live): Umgebungsvariable pro LXC
    # Phase 2 (Nexus live): GET /api/v1/config/model/{agent_role}
    return os.environ.get(
        f"MODEL_{agent_role.upper()}",
        "anthropic:claude-sonnet-4-6"  # Fallback immer Claude Sonnet
    )
```

Fallback-Regel: Wenn model_config fehlt → immer anthropic:claude-sonnet-4-6. Kein Agent darf abstürzen.
Phase 1 (jetzt): ENV-Variablen pro LXC. Phase 2 (nach Nexus): model_config Tabelle.
Gilt für: Anima-Agents, KAM-Layer, alle Paperclip-Agents, Codex Critic.

### P2 — Single Paperclip Instance
Eine Paperclip-Instanz (LXC 110). Mandantentrennung via KAM-Layer + Datenflagging.

### P3 — Thomas als Human Principal
Thomas Z. ist oberste Autorität. Review zwingend bei: Breaking Changes, Security-Änderungen, >5 Tabellen, Kundendaten direkt betroffen.

### P4 — Kein Deployment ohne Test auf LXC 110
LXC 110 = interne Test-Instanz. LXC 300+ nur nach erfolgreichem Test auf 110.

### P5 — Fokus vor Feature
Kein neues Feature bevor aktueller Pilot produktiv und stabil läuft.

---

## 3. Plattform-Architektur (5 Layer)

LAYER 0 — Kunden-Zugänge
PRIMARY: Matrix (self-hosted, .104) — DSGVO-konform
FALLBACK: Telegram (nur mit expliziter Kunden-Einwilligung)
ADD-ON: InvoiceNinja Web-Interface (je Mandant)

LAYER 1 — Verschlüsselungs-Agent (pro Mandant, lokal, kein LLM)
EINGEHEND: NER (spaCy) → Token-Mapping → Key nur im RAM → MD Chunks laden & entschlüsseln (AES-256-GCM)
AUSGEHEND: Re-encrypt → Key aus RAM löschen → Klartext an Kunde

LAYER 2 — KAM (Key Account Manager)
Modell: Mistral API (EU) → lokal wenn HW bereit
Routing, Kontext-Anreicherung, Mandant-Flagging. Sieht nie echte PII.

LAYER 3 — Paperclip (eine Instanz, shared)
LXC 110 / Branch: feature/enable-openclaw-gateway
Jarvis (CEO) → Assistent CEO → Engineering (Code Architect, Writer, Critic, DevOps, DocAgent) + ISDS Lead (Security, Compliance, License) + Research Agent
Arbeitet ausschließlich mit anonymisierten Tokens.

LAYER 4 — Datenschicht (Qdrant, self-hosted LXC 111)
public: öffentliches Wissen, alle Agents lesbar, kein Personenbezug
mandant_{id}: private Collection pro Kunde, AES-256-GCM encrypted, Key nur beim Kunden
Löschrecht: gesamte Collection auf Anfrage rückstandslos löschen, Eintrag in audit_log (nur mandant_id + Timestamp)

LAYER 5 — Nexus Billing Backend (LXC 108 → umbenennen zu nexus-billing)
API-Keys, model_config, Cost Tracking, InvoiceNinja Sync, Mandanten-Verwaltung, Qdrant Collection-Registry

### Datenfluss (Folge-Anfrage)
Kunde (Key + Anfrage)
→ Verschlüsselungs-Agent: NER → Token-Mapping → MD Chunks laden & entschlüsseln
→ KAM (Mistral): Routing + Kontext
→ Paperclip: Verarbeitung mit anonymisierten Tokens
→ Verschlüsselungs-Agent: Re-encrypt → Key löschen → Klartext an Kunde
→ async: Paperclip → Nexus: usage_log

### System-Dependencies
Sync: Enc-Agent→Nexus, Enc-Agent→Qdrant, KAM→Nexus, KAM→Paperclip, Paperclip→Nexus (model)
Async: Paperclip→Nexus (usage_log), Enc-Agent→Qdrant (upsert)

### Fallbacks
Nexus down → ENV-Variablen (P1 greift automatisch)
Qdrant down → Anfrage ablehnen, Kunde informieren
Paperclip down → HTTP 503, kein Retry
model_config missing → anthropic:claude-sonnet-4-6

### Zero-Knowledge-Garantie
SyntiqAI sieht Kundendaten nie im Klartext. Key liegt ausschließlich beim Kunden.

---

## 4. Kunden-Modelle

Modell A — Anima Hosted: SyntiqAI stellt Anima bereit. Primär Matrix, Fallback Telegram mit Einwilligung, optional InvoiceNinja Web.
Modell B — BYOA: Kunde bringt eigenen Agent, verbindet via REST API, nutzt unsere Prozesse.

### Interface DSGVO-Status
Matrix (self-hosted .104): PRIMÄR — vollständig konform
InvoiceNinja Web (self-hosted): konform
Telegram (Fallback): nur mit expliziter Einwilligung (Drittland Dubai)
REST API: geplant, konform

### Telegram-Einwilligungsregel
Kunden müssen schriftlich über Drittlandtransfer (Dubai) informiert werden und explizit einwilligen. Matrix aktiv als Alternative anbieten.

---

## 5. Verschlüsselungs-Agent

Kein LLM — deterministischer Python-Service pro Mandant.
NER: spaCy de_core_news_lg (lokal, kostenlos)
Verschlüsselung: AES-256-GCM (Authenticated Encryption)
Token-Mapping: deterministisch pro Session ([PERSON_A], [ORG_A], [AMOUNT_1])
Key: nie persistiert, nur Session-RAM, nach Response löschen (del key, gc.collect())

### MD Pool Löschkonzept
Identifier: pseudonyme mandant_id — kein Personenbezug bei SyntiqAI
Auf Löschanfrage: gesamte Qdrant Collection mandant_{id} löschen
Nachweis: Nexus audit_log (nur mandant_id + Timestamp, kein Inhalt)

---

## 6. KAM-Layer

Aktuell: Mistral API (EU, Frankreich) — kein Drittlandtransfer
Zielzustand: lokal wenn GPU-Hardware verfügbar
Migration: nur model_config Änderung (P1)
KAM-Aufgaben: Routing, Kontext-Anreicherung (Qdrant semantic search), Mandant-Flagging, Response-Qualitätsprüfung

---

## 7. Nexus Billing Backend

NICHT mehr: Dispatcher, Routing, Cell-Spawning
NEU: Billing, Konfiguration, Mandanten-Verwaltung

Bestehende DB-Migrations (11 Migrations, 8 Tabellen) bleiben gültig:
mandants, api_keys, usage_logs, billing_periods, invoices, integration_queue, cost_attribution_rules, audit_log

Neue Tabellen hinzufügen:
model_config: id, agent_role, model_string, mandant_id (nullable), updated_at
service_addons: id, mandant_id, service_type, config_json, active

---

## 8. Infrastruktur

| LXC | IP | Hostname | Funktion | Status |
|-----|-----|---------|---------|--------|
| — | .104 | matrix | Matrix Homeserver (Synapse) | Aktiv — primäres Kommunikationstool |
| 108 | .108 | nexus | Nexus Billing Backend | Aktiv, umbenennen zu nexus-billing |
| 110 | .110 | invoiceninja-test | InvoiceNinja intern/Test | Aktiv |
| 111 | .111 | qdrant | Qdrant Vector DB + Enc-Agent | Setup ausstehend |
| 300 | .150 | customer-01 | Erster Kunde (InvoiceNinja produktiv) | Setup ausstehend |
| — | .96 | openclaw | Jarvis / OpenClaw Gateway | Aktiv |
| — | .105 | nginxproxymanager | Reverse Proxy | Aktiv |
| — | .83 | homeassistant | Home Assistant | Aktiv |
| — | .82 | nas-tza | Synology Backup | Aktiv |

IP-Zonen:
.1–.10: Infrastruktur
.20–.99: Feste Heimgeräte
.100–.149: SyntiqAI intern (LXC 111 = Qdrant/Enc-Agent)
.150–.199: Kunden produktiv (LXC 300 = customer-01)
.200–.254: IoT/WLAN dynamisch

---

## 9. Autonomie-Framework & Breaking Change Definition

### Autonom erlaubt wenn ALLE erfüllt:
- Weniger als 5 Tabellen betroffen
- Kein Breaking Change
- Kein Security-Issue
- Keine Kundendaten direkt betroffen

### IMMER Breaking Change (Thomas-Review zwingend):
DB: Spalte umbenennen/löschen, Datentyp ändern, NOT NULL zu bestehender Spalte
API: Endpoint-URL ändern, Response-Format ändern, Pflichtfeld hinzufügen, Auth ändern
Verschlüsselung: Key-Format, Algorithmus, IV-Handling ändern
Mandanten-Logik: mandant_id Schema, Qdrant Collection-Naming ändern
Konfiguration: model_config Struktur, API-Key Format ändern

### KEIN Breaking Change (autonom erlaubt):
DB: Neue Spalte mit DEFAULT, neuer Index, neue Tabelle
API: Neuer optionaler Parameter, neuer Endpoint, optionale Response-Felder
Code: Refactoring, Logging, Kommentare, Tests
Dokumentation: SKILL.md, README, Inline-Docs

### API-Versionierung
/api/v1/... produktiv, /api/v2/... neue Breaking-Change Version
Deprecation: 4 Wochen Parallelbetrieb, Warning im Response-Header ab Tag 1

### IN_PROGRESS Regel
Nur setzen wenn Sub-Agent tatsächlich aktiv arbeitet — nicht bei Assignment.
```

---

## TEIL 2 — PROJEKTE SCHLIESSEN

### 2.1 Projekt "Client Relations" archivieren
Archiviere das gesamte Projekt "Client Relations" in Paperclip. Status: ARCHIVED. Closing-Kommentar: "Ersetzt durch SYNTIQAI-CORE Projekt (Nordstern-Reset 2026-04-06)."

### 2.2 Projekt "SyntiqAI" (bestehendes Hauptprojekt) archivieren
Archiviere das bestehende Hauptprojekt "SyntiqAI" in Paperclip. Status: ARCHIVED. Closing-Kommentar: "Ersetzt durch SYNTIQAI-CORE Projekt (Nordstern-Reset 2026-04-06)."

### 2.3 Einzelne Tasks schließen

Schließe folgende Tasks mit Status CLOSED und Kommentar:
"Geschlossen im Rahmen des Nordstern-Resets 2026-04-06. Konzept obsolet oder ersetzt durch SYNTIQAI-CORE. Details im Nordstern-Dokument v1.9."

**Sofort schließen — Konzept obsolet:**
- SYN-7: Paperclip UI Theme Branding
- SYN-10: Multi-Tenant Billing System (Dispatcher-Konzept)
- SYN-11: Claude Desktop MCP Integration
- SYN-13: Engineer Board Approval / Coding Cell
- SYN-17: Marketing Abteilung aufbauen
- SYN-21: Customer Portal Web-App (CR-4)
- SYN-22: Dispatcher Anbindung (CR-5)
- SYN-23: Pilot Flo onboarden (CR-6)
- SYN-24: Jarvis Memory Token-Optimierung
- SYN-26: Memory Analyse (SYN-24.2)
- SYN-27: Memory-Architektur Design (SYN-24.3)
- SYN-29: Jarvis KAIROS-Mode
- SYN-30: Jarvis AutoDream
- SYN-33: Claude.ai als MCP Review-Instanz
- SYN-34: Claude Read-Only Dashboard

**Schließen — durch neue Epics ersetzt:**
- SYN-18: KAM Template (CR-1) → ersetzt durch ENC-001 bis ENC-003
- SYN-19: Customer Onboarding Workflow (CR-2) → ersetzt durch PILOT-001 bis PILOT-005
- SYN-20: Cost Control System (CR-3) → ersetzt durch NEX-001 bis NEX-003
- SYN-32: DSGVO-Audit & PII-Pipeline → ersetzt durch DSGVO-001 bis DSGVO-007

**Schließen mit anderem Kommentar — Architektur-Design ersetzt durch Nordstern:**
- SYN-31: Billing & Dispatcher Architektur-Design
  Kommentar: "Design-Dokument ersetzt durch Nordstern v1.9. DB-Migrations (PHASE1-DB) bleiben gültig und werden in SYNTIQAI-CORE migriert."

**NICHT schließen — direkt in SYNTIQAI-CORE migrieren:**
- PHASE1-DB → wird zu MIGRATE-001
- PHASE1-PII → wird zu DSGVO-001 ff. (Inhalt übernehmen)
- SYN-16: Paperclip SKILL.md → wird zu MIGRATE-002

---

## TEIL 3 — NEUES PROJEKT ANLEGEN

### Projekt: SYNTIQAI-CORE
**Beschreibung:** Zentrales Entwicklungsprojekt für die SyntiqAI-Plattform. Ersetzt alle bisherigen Projekte. Basis: Nordstern-Architekturdokument v1.9 (2026-04-06). Vision: AI-Workforce-as-a-Service für österreichische KMUs.

---

## TEIL 4 — ALLE EPICS UND TASKS ANLEGEN

Lege folgende Struktur exakt so an. Priorität der Epics = Reihenfolge unten.

---

### EPIC 1: DSGVO & LEGAL
**Beschreibung:** Alle rechtlichen Voraussetzungen für Produktivbetrieb. Blockiert EPIC 2.
**Label:** dsgvo, legal, blocker
**Assignee Epic:** ISDS Lead

#### DSGVO-001: Datenschutzerklärung erstellen
**Assignee:** ISDS Lead | **Priorität:** KRITISCH | **Blockiert:** PILOT-003, PILOT-004
**Beschreibung:**
Datenschutzerklärung (Privacy Policy) für syntiq-ai.at nach österreichischem Recht (DSG + DSGVO) erstellen. Muss abdecken: Verantwortlicher (Thomas Z., SyntiqAI, Baden bei Wien), verarbeitete Datenkategorien, Rechtsgrundlagen (Art. 6 DSGVO), Zwecke der Verarbeitung, Drittlandtransfers (Anthropic USA via SCCs, Telegram-Fallback mit Einwilligung), Auftragsverarbeiter (Anthropic, Mistral), Betroffenenrechte (Art. 15-22), Kontakt DSB (dsb.gv.at), Speicherdauer/Löschfristen, Hosting-Infos (EU-Server). Entwurf als Markdown im Workspace speichern unter: /home/claw/.openclaw/workspace/DSGVO-001-Privacy-Policy-Entwurf.md. Dann Thomas zur Review melden.
**Abnahme:** Thomas hat reviewed und freigegeben. Dann: auf syntiq-ai.at/datenschutz veröffentlichen (Nginx Proxy Manager auf .105 konfigurieren).

#### DSGVO-002: AVV-Vorlage für Kunden
**Assignee:** ISDS Lead | **Priorität:** KRITISCH | **Blockiert:** PILOT-003
**Beschreibung:**
Standardisierte Auftragsverarbeitungsvereinbarung (AVV) nach Art. 28 DSGVO erstellen. Vorlage muss für jeden B2B-Kunden individuell angepasst werden können. Enthält: Gegenstand und Dauer, Art und Zweck der Verarbeitung, Datenkategorien und betroffene Personen, Pflichten des Auftragsverarbeiters (SyntiqAI), Liste der Subauftragsverarbeiter (Anthropic PBC USA + SCCs, Mistral AI SAS FR), Löschkonzept (MD Pool rückstandslos auf Anfrage, audit_log Nachweis), TOMs (AES-256-GCM, spaCy NER, Zero-Knowledge Key, EU-Server, SSH-Key Auth, Backup Synology). Als Markdown-Vorlage im Workspace: /home/claw/.openclaw/workspace/DSGVO-002-AVV-Vorlage.md. Thomas zur Review melden.
**Abnahme:** Thomas hat Vorlage freigegeben. Bereit für Unterzeichnung mit erstem Kunden.

#### DSGVO-003: AVV mit Anthropic abschließen
**Assignee:** ISDS Lead | **Priorität:** HOCH
**Beschreibung:**
Anthropic Data Processing Agreement (DPA) recherchieren. URL: https://www.anthropic.com/legal/data-processing-addendum — Inhalt prüfen und zusammenfassen. Standard-Vertragsklauseln (SCCs) nach Art. 46 Abs. 2 lit. c DSGVO identifizieren. Zusammenfassung der relevanten Punkte und Anweisungen für Thomas wie er die DPA abschließt: an welcher Stelle auf anthropic.com, welche Felder ausfüllen, ob Unterzeichnung oder nur Checkbox. Alles in Workspace dokumentieren: /home/claw/.openclaw/workspace/DSGVO-003-Anthropic-DPA.md
**Abnahme:** Dokument enthält alle Infos für Thomas. Thomas schließt DPA selbst ab (erfordert Account-Zugang).

#### DSGVO-004: AVV mit Mistral AI abschließen
**Assignee:** ISDS Lead | **Priorität:** HOCH
**Beschreibung:**
Mistral AI Data Processing Agreement recherchieren. URL: https://mistral.ai/terms — DPA-Sektion finden. EU-intern (Frankreich) — keine SCCs erforderlich. Zusammenfassung wie Thomas die DPA abschließt. Workspace: /home/claw/.openclaw/workspace/DSGVO-004-Mistral-DPA.md
**Abnahme:** Dokument vollständig. Thomas schließt DPA ab.

#### DSGVO-005: Incident Response Plan
**Assignee:** ISDS Lead | **Priorität:** HOCH
**Beschreibung:**
Incident Response Plan für Datenpannen nach Art. 33/34 DSGVO erstellen. Enthält: Definition meldepflichtiger Vorfall, interne Eskalationskette (Entdeckung → Jarvis meldet sofort an Thomas), 72h-Meldefrist an DSB (dsb.gv.at, +43 1 52 152-0, meldung@dsb.gv.at), Meldevorlage an DSB (Felder: wer, was, wann, wie viele Betroffene, Maßnahmen), Benachrichtigungsvorlage für betroffene Kunden (Art. 34), Nachsorge (Ursache beheben, Dokumentation). Workspace: /home/claw/.openclaw/workspace/DSGVO-005-Incident-Response-Plan.md. Thomas zur Review.
**Abnahme:** Thomas hat Plan genehmigt.

#### DSGVO-006: Verzeichnis der Verarbeitungstätigkeiten (VVT)
**Assignee:** ISDS Lead | **Priorität:** HOCH
**Beschreibung:**
VVT nach Art. 30 DSGVO anlegen. Pro Verarbeitungstätigkeit: Name, Zweck, Rechtsgrundlage, Datenkategorien, betroffene Personen, Empfänger, Drittlandtransfers, Löschfristen. Tätigkeiten: (1) Kundenkommunikation via Matrix, (2) KI-Verarbeitung via Anthropic Claude, (3) KAM via Mistral AI, (4) Billing via Nexus/InvoiceNinja, (5) Datenspeicherung Qdrant/MySQL, (6) Backup Synology, (7) Telegram-Fallback (mit Hinweis: Drittland, Einwilligung erforderlich). Als lebendes Dokument: /home/claw/.openclaw/workspace/DSGVO-006-VVT.md. Laufend aktualisieren bei neuen Verarbeitungen.
**Abnahme:** VVT vollständig, Thomas reviewed.

#### DSGVO-007: Telegram Einwilligungsdokument
**Assignee:** ISDS Lead | **Priorität:** MITTEL
**Beschreibung:**
Informierte Einwilligungserklärung für Telegram-Fallback nach Art. 49 Abs. 1 lit. a DSGVO erstellen. Muss klar erklären: was Telegram ist, warum Dubai ein Drittland ist, welche Daten übertragen werden, dass Matrix als datenschutzkonforme Alternative verfügbar ist, dass die Einwilligung jederzeit widerrufbar ist. Zweisprachig: Deutsch + Englisch. Workspace: /home/claw/.openclaw/workspace/DSGVO-007-Telegram-Einwilligung.md. Thomas zur Review — wird Jurist zur Bestätigung vorgelegt.
**Abnahme:** Entwurf erstellt, Thomas reviewed.

---

### EPIC 2: PILOT-PRODUKTION
**Beschreibung:** Technischer Aufbau für ersten produktiven Kunden. Startet ERST wenn EPIC 1 vollständig abgeschlossen und von Thomas freigegeben.
**Label:** pilot, produktion, infrastructure
**Assignee Epic:** DevOps Agent
**Abhängigkeit:** ALLE Tasks aus EPIC 1 abgeschlossen + Thomas-Freigabe

#### PILOT-001: LXC 300 aufsetzen (customer-01)
**Assignee:** DevOps Agent | **Priorität:** HOCH (nach DSGVO-Freigabe)
**Beschreibung:**
Neuen Proxmox LXC auf pve-zat anlegen:
- LXC ID: 300
- Hostname: customer-01
- Template: Ubuntu 22.04 LTS
- RAM: 2048 MB
- CPU: 2 Cores
- Disk: 20 GB (local-lvm)
- Netzwerk: vmbr0, IP 192.168.178.150/24, Gateway 192.168.178.1
- Unprivileged: yes

Nach Start: MAC-Adresse auslesen (ip link show eth0). Thomas melden: "Bitte DHCP-Reservierung in FritzBox für MAC [MAC] auf IP 192.168.178.150 anlegen." Warten auf Bestätigung.

Nach FritzBox-Reservierung: Docker + Docker Compose installieren. SSH-Key von openclaw (.96) auf LXC 300 autorisieren (/root/.ssh/authorized_keys). Proxmox Backup-Job anlegen: täglich 02:00, Target Synology NAS (192.168.178.82), Retention 7 Tage, Snapshot-Mode.
**Abnahme:** LXC läuft, SSH von openclaw funktioniert (ssh root@192.168.178.150), Backup-Job aktiv.

#### PILOT-002: InvoiceNinja produktiv deployen
**Assignee:** DevOps Agent | **Priorität:** HOCH | **Abhängigkeit:** PILOT-001
**Beschreibung:**
InvoiceNinja 5 via Docker Compose auf LXC 300 deployen. Docker Compose Datei unter /opt/invoiceninja/docker-compose.yml. Services: invoiceninja (invoiceninja/invoiceninja:5) + db (mysql:8). APP_URL: http://192.168.178.150 (zunächst intern). Sichere Passwörter für DB generieren (pwgen 32 1). APP_KEY generieren (php artisan key:generate). Volumes: ./storage, ./public, ./mysql. Port: 80:80. Nach Start: InvoiceNinja Setup-Wizard durchlaufen, Admin-Account anlegen. API-Token erstellen: Settings → API Tokens → New Token → Name: "jarvis-agent". Token sicher in OpenClaw Konfiguration hinterlegen (NICHT in Plaintext-Dateien, NICHT im Workspace). Test: curl -H "X-API-Token: {token}" http://192.168.178.150/api/v1/clients → muss HTTP 200 zurückgeben.
**Abnahme:** InvoiceNinja erreichbar, API-Token funktioniert, Admin-Account erstellt.

#### PILOT-003: Matrix Bot implementieren
**Assignee:** Code Writer | **Priorität:** HOCH | **Abhängigkeit:** PILOT-001 + DSGVO-001 freigegeben
**Beschreibung:**
Matrix Bot als Python-Service auf LXC 111 implementieren (wird mit Qdrant/Enc-Agent geteilt — LXC 111 muss zuerst existieren, ggf. minimal aufsetzen). Bot-Account auf Matrix Homeserver (.104) anlegen: @anima:syntiq-ai.at (oder verfügbare Domain). Matrix SDK: matrix-nio (Python). Bot-Funktionen: Nachrichten empfangen → an Anima-Agent (via Paperclip API) weiterleiten → Response zurück in Matrix-Raum senden. Pro Kunde: eigener Matrix-Raum, eigene mandant_id. Konfiguration: Matrix Homeserver URL, Bot-Credentials — sicher in ENV-Variablen, nicht in Code. Kein Telegram-Fallback implementieren bis DSGVO-007 von Thomas freigegeben.

Matrix Raum-Naming Standard (zwingend einhalten):
- Arbeitsraum pro Kunde: #work-{mandant_id_kurz}:syntiq-ai.at (z.B. #work-abc123:syntiq-ai.at)
- Privater Key-Share Raum: #secure-{mandant_id_kurz}:syntiq-ai.at (nur für Share-1-Übergabe, danach schliessen)
- Interner Test-Raum: #test-internal:syntiq-ai.at
- mandant_id_kurz = erste 6 Zeichen der mandant_id
Alle Räume: E2EE aktiviert, Anima als Bot-Member, Thomas als Admin.
**Abnahme:** Nachricht im Matrix-Raum → Bot leitet an Anima weiter → Anima antwortet im selben Raum.

#### PILOT-004: Anima-Agent konfigurieren
**Assignee:** Assistent CEO | **Priorität:** HOCH | **Abhängigkeit:** PILOT-003
**Beschreibung:**
Anima-Agent in Paperclip konfigurieren. Modell: get_model("anima") — ENV-Variable MODEL_ANIMA=anthropic:claude-sonnet-4-6 auf LXC 110 und LXC 300 setzen. Kein Hard-Coding (P1). System-Prompt: Anima ist ein freundlicher, professioneller KI-Assistent für österreichische KMUs. Kommuniziert auf Deutsch (außer Kunde schreibt Englisch). Spezialisiert auf Geschäftsprozesse: Angebote, Rechnungen, administrative Aufgaben. Initiale Skills: (1) Angebotserstellung via InvoiceNinja API — Kunde beschreibt Leistung → Anima erstellt Angebot → Vorschau zurück → bei Bestätigung senden. (2) Rechnungserstellung analog. Bei unklaren Anfragen: nachfragen, nicht raten. Bei Anfragen außerhalb der Skills: ehrlich kommunizieren was noch nicht möglich ist.
**Abnahme:** Anima empfängt Anfrage via Matrix, erstellt Angebot in InvoiceNinja, sendet Vorschau zurück, wartet auf Bestätigung.

#### PILOT-005: End-to-End Flow testen und dokumentieren
**Assignee:** Code Writer | **Priorität:** HOCH | **Abhängigkeit:** PILOT-004
**Beschreibung:**
Vollständigen Flow testen. Happy Path: Matrix-Nachricht "Erstelle ein Angebot für Kunde Mustermann, 8 Stunden Support à 95 Euro" → Anima verarbeitet → InvoiceNinja Angebot erstellt → PDF-Vorschau in Matrix → Bestätigung → Angebot versendet. Fehlerszenarien testen: (1) Fehlende Kundendaten → Anima fragt nach. (2) InvoiceNinja nicht erreichbar → freundliche Fehlermeldung, kein Crash. (3) Ungültige Anfrage → Anima erklärt was sie kann. (4) Anima gibt falsche Daten zurück → Validierung. Alle Ergebnisse dokumentieren: /home/claw/.openclaw/workspace/PILOT-005-Testergebnisse.md. Thomas melden wenn abgeschlossen.
**Abnahme:** Alle Tests bestanden, Ergebnisse dokumentiert, Thomas hat freigegeben.

---

### EPIC 3: NEXUS BILLING
**Beschreibung:** Nexus als Billing & Configuration Backend aufbauen.
**Label:** nexus, billing, backend
**Assignee Epic:** Code Writer
**Abhängigkeit:** EPIC 2 abgeschlossen

#### NEX-001: model_config ENV-Setup (P1 Implementation)
**Assignee:** DevOps Agent | **Priorität:** MITTEL
**Beschreibung:**
ENV-Variablen für Model-Agnostizität (P1) auf allen relevanten LXCs setzen. Auf openclaw (.96): MODEL_ANIMA=anthropic:claude-sonnet-4-6, MODEL_KAM=mistral:mistral-small-latest, MODEL_PAPERCLIP=anthropic:claude-sonnet-4-6, MODEL_CODEX_CRITIC=mistral:mistral-small-latest. Auf LXC 300: gleiche Variablen. get_model() als Python-Modul implementieren (/opt/syntiqai/lib/model_config.py) und in alle Agents einbinden. Test: MODEL_ANIMA auf mistral:mistral-small-latest ändern → Anima nutzt Mistral ohne Code-Change.
**Abnahme:** Modellwechsel via ENV funktioniert. Test erfolgreich.

#### NEX-002: Nexus umbenennen und neue Tabellen
**Assignee:** DevOps Agent + Code Writer | **Priorität:** MITTEL | **Thomas-Review: JA (Breaking Change Hostname)**
**Beschreibung:**
Hostname LXC 108 von "nexus" auf "nexus-billing" ändern. Thomas fragen bevor Umbenennung — Hostname-Änderung könnte bestehende Verbindungen brechen. Nach Thomas-Freigabe: Hostname ändern, alle internen Referenzen auf neuen Hostname aktualisieren. Neue DB-Migrations schreiben (KEIN Breaking Change — neue Tabellen): Migration für model_config (id UUID, agent_role VARCHAR, model_string VARCHAR, mandant_id UUID nullable, updated_at TIMESTAMP). Migration für service_addons (id UUID, mandant_id UUID, service_type VARCHAR, config_json JSON, active BOOLEAN, created_at TIMESTAMP). Forward + Reverse Migrations testen.
**Abnahme:** Nexus-Billing läuft unter neuem Hostname. Neue Tabellen migriert. Forward + Rollback getestet.

#### NEX-003: InvoiceNinja Sync
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** NEX-002
**Beschreibung:**
Async Job der täglich 03:00 Uhr InvoiceNinja (LXC 300) mit Nexus-Billing synchronisiert. InvoiceNinja API: GET /api/v1/invoices?updated_at={last_sync_timestamp}. Neue Invoices → usage_logs Tabelle eintragen. Mandant-Attribution via mandant_id (aus InvoiceNinja Client-Tags oder Custom Fields). Kein Breaking Change an bestehenden Tabellen. Job-Status in audit_log. Bei Fehler: Job retry 3x, dann Thomas via Matrix benachrichtigen.
**Abnahme:** Nach 24h erster Sync, usage_logs enthält korrekte Einträge.

---

### EPIC 4: VERSCHLÜSSELUNGS-AGENT & MD POOL
**Beschreibung:** Zero-Knowledge Datenschicht implementieren.
**Label:** encryption, dsgvo, qdrant, security
**Assignee Epic:** Code Writer
**Abhängigkeit:** EPIC 2 abgeschlossen

#### ENC-001: LXC 111 aufsetzen (qdrant + enc-agent)
**Assignee:** DevOps Agent | **Priorität:** MITTEL
**Beschreibung:**
Proxmox LXC anlegen:
- LXC ID: 111
- Hostname: qdrant-enc
- IP: 192.168.178.111/24
- Template: Ubuntu 22.04 LTS
- RAM: 4096 MB (Qdrant braucht etwas mehr)
- CPU: 2 Cores
- Disk: 40 GB (Qdrant-Daten können wachsen)

MAC auslesen, Thomas für FritzBox-Reservierung (.111) melden. Docker installieren. SSH-Key von openclaw autorisieren. Backup-Job: täglich 02:30, Synology NAS, 7 Tage.
**Abnahme:** LXC läuft, SSH von openclaw funktioniert, Backup aktiv.

#### ENC-002: Qdrant deployen
**Assignee:** DevOps Agent | **Priorität:** MITTEL | **Abhängigkeit:** ENC-001
**Beschreibung:**
Qdrant via Docker auf LXC 111 deployen. Docker Compose: qdrant/qdrant:latest, Port 6333, Volume ./qdrant_storage. Zwei Collections anlegen: "public" (für geteiltes Wissen, keine Verschlüsselung) und Dokumentation des Collection-Schemas für mandant_{id} (wird dynamisch pro Kunde angelegt). REST-Endpoint testen: GET http://192.168.178.111:6333/collections → HTTP 200.
**Abnahme:** Qdrant läuft, public Collection angelegt, API erreichbar von openclaw.

#### ENC-003: spaCy NER Service implementieren
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ENC-001
**Beschreibung:**
Python FastAPI Service auf LXC 111 implementieren. spaCy mit deutschem Modell: python -m spacy download de_core_news_lg. Endpoints: POST /anonymize: Input: {text: str, session_id: str} → Output: {anonymized_text: str, mapping: dict}. POST /deanonymize: Input: {text: str, mapping: dict} → Output: {text: str}. Token-Mapping: deterministisch pro session_id, im RAM gehalten. Mapping wird NICHT persistiert. Erkannte Entities: PER, ORG, LOC, MONEY, DATE. Mapping-Format: PERSON_A, ORG_A, LOC_A, AMOUNT_1, DATE_1 etc. Test: "Erstelle Angebot für Max Mustermann, Firma TechCorp, 1000 Euro" → "Erstelle Angebot für [PERSON_A], [ORG_A], [AMOUNT_1]".
**Abnahme:** Anonymisierung + Deanonymisierung korrekt. Mapping nicht persistiert. Service als systemd Service konfiguriert.

#### ENC-004: AES-256-GCM Verschlüsselungsmodul
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ENC-003
**Beschreibung:**
Python-Modul für AES-256-GCM Verschlüsselung implementieren (/opt/syntiqai/lib/encryption.py). Funktionen: encrypt(plaintext: str, key: bytes) → {ciphertext: bytes, nonce: bytes, tag: bytes}. decrypt(ciphertext: bytes, nonce: bytes, tag: bytes, key: bytes) → str. Key kommt immer als Session-Parameter — wird NIEMALS gespeichert oder geloggt. Nach Verwendung: del key, gc.collect(). Kein Logging von Keys, Plaintext oder verschlüsselten Inhalten. Library: pycryptodome (AES.MODE_GCM). Integrations-Test mit Qdrant: encrypt → in Qdrant speichern → aus Qdrant laden → decrypt → original.
**Abnahme:** Encrypt/Decrypt korrekt. Key nach Session nicht mehr im Memory (verifiziert via gc.get_referrers Test). Integration mit Qdrant funktioniert.

#### ENC-005: Lösch-Endpoint implementieren
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ENC-004
**Beschreibung:**
DELETE /mandant/{mandant_id} Endpoint im FastAPI Service. Ablauf: (1) Qdrant Collection mandant_{mandant_id} vollständig löschen. (2) Eintrag in Nexus audit_log schreiben: {action: "MANDANT_DELETED", mandant_id: str, timestamp: ISO8601, deleted_by: "system"} — KEIN Inhalt, nur Metadaten. (3) HTTP 200 + Bestätigung zurückgeben. Bei nicht-existenter Collection: HTTP 404. Wichtig: Kein Backup der Collection vor Löschung — rückstandslose Löschung ist Pflicht (Art. 17 DSGVO).
**Abnahme:** Löschung funktioniert, audit_log enthält Eintrag, Collection ist danach nicht mehr vorhanden (verifiziert).

---

### EPIC 5: MARKETING & AUSSENDARSTELLUNG
**Beschreibung:** Läuft parallel — unabhängig von allen anderen Epics.
**Label:** marketing, website, brand
**Assignee Epic:** Research Agent

#### MKT-001: Logo Canva — Vorbereitung und Varianten
**Assignee:** Research Agent | **Priorität:** HOCH
**Beschreibung:**
Logo "Syntiq." in Canva liegt unter https://www.canva.com/d/JxDn824yxrQRTK8. Jarvis kann das Logo nicht direkt bearbeiten — Thomas macht das manuell. Jarvis bereitet vor: (1) Stil-Briefing für Thomas: Dark Navy (#1B3A5C) für Schriftzug, Cyan (#00BCD4 oder ähnlich) für Punkt über q und "AI". Empfohlene Schriftarten für "Syntiq.": recherchieren (modern, tech, seriös — z.B. Space Grotesk, Inter, Syne). (2) Export-Anleitung: SVG transparent (für Web), PNG 512x512 transparent (für Apps), PNG 1920x1080 auf dunkel (für Präsentationen). (3) Varianten-Vorschlag: mit/ohne "AI" Suffix, horizontal/quadratisch. Alles in Workspace: /home/claw/.openclaw/workspace/MKT-001-Logo-Briefing.md. Thomas melden.
**Abnahme:** Briefing-Dokument erstellt, Thomas informiert.

#### MKT-002: Website-Konzept und Grundstruktur
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** MKT-001 (Logo muss final sein)
**Beschreibung:**
Next.js + Vercel + Sanity CMS Stack (bereits entschieden). GitHub Repo: syntiqAI/syntiq-ai-website. Seiten: (1) Landing Page: Hero (AI-Workforce-as-a-Service), Features (3 Hauptvorteile), Wie es funktioniert (3 Schritte), CTA (Kontakt). (2) Leistungen: Anima Hosted, BYOA, Add-ons. (3) Über uns: Vision, Team (Thomas). (4) Kontakt: Formular (kein Backend-Call — mailto vorerst). (5) Datenschutz: Platzhalter mit Link auf DSGVO-001 Inhalt. (6) Impressum: österreichisches Pflichtimpressum (Thomas befüllt). Design: SyntiqAI Brand-Farben (Navy + Cyan), modern, technisch, seriös. Keine Preisangaben. Deploy auf Vercel. DNS auf syntiq-ai.at wenn A1-Problem gelöst.
**Abnahme:** Website auf Vercel deployed, alle Seiten erreichbar, responsiv, Datenschutz-Platzhalter vorhanden.

#### MKT-003: Kanal-Research für österreichische KMUs
**Assignee:** Research Agent | **Priorität:** NIEDRIG
**Beschreibung:**
Research: Welche Kanäle nutzen österreichische KMU-Entscheider und IT-Verantwortliche? LinkedIn (DACH), Xing, WKO-Netzwerk (wko.at), lokale Wirtschaftskammer NÖ, IT-Branchenverbände (ISPA), Fachmedien (futurezone.at, heise.de/at). Ergebnis: Report mit Kanal-Empfehlung, geschätzter Reichweite, Aufwand, Content-Ideen für ersten Monat. Workspace: /home/claw/.openclaw/workspace/MKT-003-Kanal-Research.md. Kein Output nach außen ohne Thomas-Freigabe.
**Abnahme:** Research-Report vollständig, Thomas reviewed.

---

### MIGRIERTE TASKS

#### MIGRATE-001: PHASE1-DB Migrations abschließen
**Assignee:** Code Writer (läuft weiter) | **Priorität:** MITTEL
**Beschreibung:**
Bestehende DB-Migrations (11 Migrations, 8 Tabellen: mandants, api_keys, usage_logs, billing_periods, invoices, integration_queue, cost_attribution_rules, audit_log) für Nexus-Billing finalisieren. Neue Migrations aus NEX-002 hinzufügen (model_config, service_addons). Alle Migrations forward + reverse auf Testdatenbank durchlaufen lassen. Dokumentation der Migrations aktualisieren.
**Abnahme:** Alle Migrations fehlerfrei. Rollback funktioniert. Getestet auf LXC 108.

#### MIGRATE-002: Paperclip SKILL.md fertigstellen (SYN-16)
**Assignee:** Documentation Agent (läuft weiter) | **Priorität:** NIEDRIG
**Beschreibung:**
Bestehende Paperclip SKILL.md (API Reference & Best Practices) fertigstellen. Neu dokumentieren: Verschlüsselungs-Agent Integration (wie Paperclip-Agents damit interagieren), Matrix Bot Anbindung, model_config Pattern (P1 Pflicht-Pattern), Mandanten-Isolation in Paperclip, Breaking Change Definition (aus Nordstern Sektion 9). Bestehende Dokumentation (50+ Endpoints) bleibt erhalten.
**Abnahme:** SKILL.md vollständig, alle neuen Architekturkomponenten dokumentiert, Thomas reviewed.

---

## TEIL 5 — ABSCHLUSS-AKTIONEN

## TEIL 5 — ABSCHLUSS-AKTIONEN & LAUFENDE PROJEKTSTEUERUNG

### 5.1 Workspace aufräumen
Lösche oder archiviere veraltete Dokumente im Workspace die durch Nordstern ersetzt wurden:
- Alle alten Architektur-Dokumente vor dem 2026-04-06
- Alle Dispatcher-bezogenen Design-Dokumente
- Alte TAGESLOG-Einträge die auf geschlossene Tasks verweisen: nicht löschen, als ARCHIVIERT kennzeichnen

Behalte:
- OPERATIONS-HANDBOOK.md (weiterhin gültig)
- TAGESLOG-2026-04-05.md (historisch)
- SYNTIQAI-NORDSTERN-2026-04-06.md (neu — Hauptreferenz)
- Alle neuen Task-Dokumente aus EPIC 1

### 5.2 Tageslog für heute anlegen
Erstelle TAGESLOG-2026-04-06.md im Workspace mit:
- Nordstern-Reset dokumentiert
- Alle geschlossenen Tasks gelistet
- Neue Projektstruktur SYNTIQAI-CORE beschrieben
- Nächste offene Aktionen für Thomas klar benannt

### 5.3 Bestätigung an Thomas senden
Sende via Matrix (primär) oder Telegram (Fallback):

```
✅ NORDSTERN-RESET ABGESCHLOSSEN

📁 Projekte archiviert:
• SyntiqAI (alt)
• Client Relations

🔴 Tasks geschlossen: 20
📋 Neues Projekt: SYNTIQAI-CORE
📊 Epics angelegt: 13
✅ Tasks angelegt: 57+

🚦 Aktueller Status:
• EPIC 1 (DSGVO): IN ARBEIT — ISDS Lead hat gestartet
• EPIC 2 (Pilot): BLOCKIERT auf EPIC 1
• EPIC 3-4: BLOCKIERT auf EPIC 2
• EPIC 5 (Marketing): AKTIV — parallel, kein Blocker
• EPIC 6 (Admin Panel): BLOCKIERT auf EPIC 2+3
• EPIC 7 (Support-Session): BLOCKIERT auf EPIC 4
• EPIC 8 (Kunden-Dashboard): BLOCKIERT auf EPIC 3 + A1 Port-Lösung
• EPIC 9 (Key-Escrow): BLOCKIERT auf EPIC 4
• EPIC 10 (Monitoring): AKTIV — MON-001 startet sofort
• EPIC 11 (Rechtliches): AKTIV — deine Tasks warten auf dich
• EPIC 12 (BYOA/Dev): BLOCKIERT auf EPIC 3
• EPIC 13 (Testing): BLOCKIERT auf EPIC 4

📋 DEINE OFFENEN PUNKTE (nur du kannst das):
1. ⏳ DSGVO-001: Privacy Policy reviewen — warte auf ISDS Lead Entwurf
2. ⏳ DSGVO-002: AVV-Vorlage reviewen — warte auf ISDS Lead Entwurf
3. 🔧 FritzBox: DHCP .150 + .111 nach MAC-Auslese von mir
4. 🔧 Anthropic DPA abschließen (DSGVO-003 Anleitung kommt von mir)
5. 🔧 Mistral DPA abschließen (DSGVO-004 Anleitung kommt von mir)
6. 🎨 Logo Canva fertigstellen (MKT-001 Briefing kommt von mir)
7. ✅/❌ NEX-002 Hostname-Änderung genehmigen wenn ich frage
8. 🔒 KEY-004: Tresor-Lösung entscheiden
9. 💶 LEGAL-002: Preismodell festlegen
10. 🏢 LEGAL-003: Gewerbeanmeldung prüfen
11. ⚖️ LEGAL-004: Juristen beauftragen (EU AI Act + DSGVO-Paper liegt bereit)
12. 📊 MON-004: RTO/RPO Werte genehmigen

Ich schicke dir Reminder wenn etwas zu lange offen bleibt.
Ersten Wochenbericht bekommst du nächsten Montag 08:00. 💪
```

### 5.4 Sofort parallel starten (ohne auf Thomas zu warten)
Diese Tasks kannst du sofort beginnen während EPIC 1 läuft:
- **ISDS Lead:** DSGVO-001, DSGVO-002, DSGVO-003, DSGVO-004, DSGVO-005, DSGVO-006
- **DevOps Agent:** MON-001 (Uptime Kuma, sobald LXC 111 bereit)
- **Research Agent:** MKT-001 (Logo Briefing), MKT-003 (Kanal-Research)
- **Documentation Agent:** MIGRATE-002 (SKILL.md)
- **Code Writer:** MIGRATE-001 (DB Migrations)

### 5.5 Laufende Projektsteuerung (dauerhaft, deine Verantwortung)
Du prüfst täglich:
- Haben alle IN_PROGRESS Tasks einen aktiv arbeitenden Agent?
- Gibt es Tasks die seit >2 Tagen nicht voranschreiten ohne guten Grund?
- Hat Thomas offene Punkte die älter als 2 Tage sind? → Reminder senden
- Kann ein blockierter Epic nun starten weil seine Abhängigkeit erfüllt ist?

Du dokumentierst täglich:
- TAGESLOG-[DATUM].md im Workspace: was wurde heute gemacht, was ist offen, was blockiert

Du eskalierst sofort an Thomas wenn:
- Ein Security-Issue auftaucht
- Ein DSGVO-relevantes Problem entdeckt wird
- Ein Produktiv-System (LXC 300 oder höher) down ist
- Ein Kunde ein Problem meldet

---

### EPIC 6: SYNTIQAI ADMIN PANEL
**Beschreibung:** Einheitliches internes Dashboard für Thomas. Gibt Überblick über Kunden, Kosten, System-Status und Anima-Templates. Läuft intern auf LXC 108 (Nexus-Billing). Kein öffentlicher Zugang.
**Label:** admin, dashboard, internal
**Assignee Epic:** Code Writer
**Abhängigkeit:** EPIC 2 + EPIC 3 abgeschlossen (Nexus-Billing muss Daten haben)

#### ADMIN-001: Admin Panel Grundstruktur
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** NEX-002
**Beschreibung:**
React Single-Page-App auf LXC 108 (Nexus-Billing) unter Port 8080. Nur intern erreichbar (192.168.178.108:8080), kein Nginx-Proxy nach außen. Auth: einfaches API-Key Login (kein OAuth nötig — nur Thomas nutzt das). Grundstruktur: vier Tabs (Kunden, Templates, System, Onboarding). Daten kommen via REST aus Nexus-Billing API. Dark-Theme passend zu SyntiqAI Brand (Navy + Cyan). Keine externen CDN-Abhängigkeiten — alles lokal gebundled.
**Abnahme:** Admin Panel erreichbar unter 192.168.178.108:8080, Login funktioniert, vier Tabs vorhanden.

#### ADMIN-002: Kunden-Übersicht
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ADMIN-001
**Beschreibung:**
Tab "Kunden" implementieren. Tabelle mit: Kundenname, mandant_id, Status (aktiv/onboarding/pausiert), Anima-Template, letzter Kontakt (Timestamp letzte Matrix-Nachricht), API-Kosten diese Woche, API-Kosten dieser Monat, Direktlink zum Matrix-Raum, Aktionen-Button. Aktionen pro Kunde: Status ändern, Template wechseln (triggert Jarvis via API), Support-Session starten (aus EPIC 7), Qdrant Collection Größe anzeigen. Datenquelle: Nexus mandants + usage_logs Tabellen. Aktualisierung: alle 60 Sekunden automatisch.

**Infrastruktur-Ansicht pro Kunde (aufklappbar):**
Jeder Kunde-Eintrag hat einen "Infrastruktur" Expand-Button. Aufgeklappt zeigt er: Tabelle aller LXCs/Container die diesem Mandanten zugeordnet sind (aus neuer Nexus-Tabelle mandant_infrastructure). Spalten: LXC-ID, Hostname, IP, Service-Typ (InvoiceNinja/Matrix-Bot/EncAgent/Custom), Status (grün/gelb/rot via HTTP-Check), CPU-Last (via Proxmox API falls verfügbar), letzter Restart, Direktlink zu Proxmox-Console (https://192.168.178.2:8006). Zuordnung erfolgt beim Onboarding automatisch durch Jarvis. Neue Nexus-Tabelle: mandant_infrastructure (id, mandant_id, lxc_id, hostname, ip, service_type, notes, active). Migration kein Breaking Change — neue Tabelle.
**Abnahme:** Alle aktiven Mandanten sichtbar. Infrastruktur-Ansicht zeigt korrekte LXCs pro Kunde. Status-Check funktioniert.

#### ADMIN-003: Anima-Template Verwaltung
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ADMIN-001
**Beschreibung:**
Tab "Templates" implementieren. Templates sind YAML/JSON-Dateien die Anima-Konfiguration definieren: Modell (via get_model()), System-Prompt, aktive Skills, InvoiceNinja-Integration ja/nein, Sprache, Antwort-Stil. Template-Liste anzeigen: Name, Beschreibung, wie viele Kunden nutzen es, letzte Änderung. Template anzeigen (read-only im Browser). Template bearbeiten: öffnet Editor (CodeMirror oder ähnlich), Speichern triggert Jarvis via Matrix: "Neues Template [NAME] deployen — bitte bestätigen." Thomas muss in Matrix bestätigen bevor Deploy. Templates gespeichert in /home/claw/.openclaw/workspace/templates/.
**Abnahme:** Templates sichtbar, bearbeitbar, Deploy-Flow via Matrix-Bestätigung funktioniert.

#### ADMIN-004: System-Status Dashboard
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ADMIN-001
**Beschreibung:**
Tab "System" implementieren. Zeigt in Echtzeit: LXC-Status (ping + HTTP-Check): matrix (.104), nexus-billing (.108), invoiceninja-test (.110), qdrant-enc (.111), customer-01 (.150), openclaw (.96), nginxproxy (.105). Paperclip Agent-Status: alle Agents mit aktuellem Status (idle/working/error). Qdrant: Collections mit Größe in MB pro Mandant. Nexus: letzter Billing-Sync Timestamp, Fehler. API-Kosten gesamt heute. Farb-Kodierung: grün/gelb/rot. Auto-Refresh alle 30 Sekunden. Bei rot: Push-Benachrichtigung via Matrix an Thomas.
**Abnahme:** Alle Systeme sichtbar mit korrektem Status. Alert via Matrix bei Fehler funktioniert.

#### ADMIN-005: Onboarding-Flow UI
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ADMIN-002 + ADMIN-003
**Beschreibung:**
Tab "Onboarding" implementieren. Formular: Kundenname, Firma, E-Mail (optional), Plan (Anima Hosted / BYOA), Template auswählen (Dropdown aus ADMIN-003), InvoiceNinja aktivieren (Checkbox), Abrechnungsmodell (Prepaid/Postpaid), Notizen. "Kunden anlegen" Button triggert Jarvis via Paperclip API.

Jarvis führt automatisch aus: mandant_id generieren, Matrix-Arbeitsraum anlegen (#work-{6-Zeichen-ID}:syntiq-ai.at), Anima-Template deployen, Qdrant Collection mandant_{id} anlegen, InvoiceNinja Kunde anlegen (wenn aktiviert), mandant_infrastructure Eintrag anlegen.

WICHTIG: Key-Generierung und Share-Ausgabe laufen nach KEY-002 (6-Schritte-Flow). Admin Panel zeigt nach Kunden-Anlage die Share-PDFs zum Download + Checkliste für Thomas (4 Häkchen). Onboarding gilt erst als abgeschlossen wenn alle Häkchen gesetzt + Codewort eingetragen. Bis dahin: Status bleibt "onboarding", Anima ist für Kunden nicht erreichbar.
**Abnahme:** Formular ausgefüllt → Jarvis legt Infrastruktur an → 3 Share-PDFs generiert → Thomas druckt + verteilt → Checkliste vollständig → Status wechselt auf "aktiv".

---

### EPIC 7: SUPPORT-ZUGANG (BREAK-GLASS ACCESS)
**Beschreibung:** Temporärer, zeitlich begrenzter Support-Zugang auf Kundendaten. Kunde erteilt explizite Einwilligung, alle Aktionen werden protokolliert, Kunde bekommt vollständiges Protokoll nach Session-Ende.
**Label:** support, security, dsgvo, audit
**Assignee Epic:** Code Writer + ISDS Lead
**Abhängigkeit:** EPIC 4 abgeschlossen (Enc-Agent + Qdrant laufen)

#### SUPPORT-001: Support-Session Datenmodell
**Assignee:** Code Writer | **Priorität:** MITTEL | **Thomas-Review: JA (neue Tabelle)**
**Beschreibung:**
Neue Tabelle in Nexus-Billing: support_sessions. Felder: id (UUID), mandant_id (UUID), requested_by (VARCHAR — "thomas" oder Agent-Name), granted_at (TIMESTAMP), expires_at (TIMESTAMP), duration_minutes (INT — 60/240/1440), status (ENUM: pending/active/expired/revoked), approved_agents (JSON-Array — Liste der Agents die Zugriff haben), session_token (VARCHAR — einmaliger Token, gehasht), created_at (TIMESTAMP). Neue Tabelle: support_session_actions. Felder: id, session_id, action_type (VARCHAR), action_detail (TEXT — was wurde gemacht), actor (VARCHAR — thomas/jarvis/agent-name), timestamp. Migration forward + reverse schreiben. Thomas-Review vor Migration.
**Abnahme:** Tabellen angelegt, Migration getestet, Rollback funktioniert.

#### SUPPORT-002: Support-Session Flow via Matrix
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** SUPPORT-001
**Beschreibung:**
Jarvis-Skill implementieren: Support-Anfrage starten. Flow:
(1) Kunde schreibt in Matrix-Raum: "Ich brauche Support" oder "Ich habe ein Problem"
(2) Anima erkennt Support-Bedarf, leitet an Jarvis weiter
(3) Jarvis schreibt Kunden an: "Für die Fehlersuche benötige ich temporären Zugang zu deinen anonymisierten Daten. Wie lange soll der Zugang gelten? 1h / 4h / 24h"
(4) Kunde antwortet mit Dauer
(5) Jarvis: "Bitte bestätige: Ich erteile SyntiqAI temporären Zugang für [DAUER]. Zugang gilt bis [ZEITSTEMPEL]. Alle Aktionen werden protokolliert. Mit 'JA BESTÄTIGEN' zustimmen."
(6) Kunde: "JA BESTÄTIGEN"
(7) Jarvis: session_token generieren, Support-Session in DB anlegen, Thomas via Matrix benachrichtigen: "Support-Session für Mandant [ID] aktiv bis [ZEITSTEMPEL]. Token: [TOKEN]"
(8) Kunde bekommt Bestätigung: "Support-Zugang aktiviert bis [ZEITSTEMPEL]. Du erhältst danach ein vollständiges Protokoll."
**Abnahme:** Vollständiger Flow funktioniert, Session in DB angelegt, Thomas benachrichtigt.

#### SUPPORT-003: Temporäre Key-Freigabe Mechanismus
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** SUPPORT-002
**Beschreibung:**
Während aktiver Support-Session: Kunde übergibt seinen Key via Matrix (verschlüsselte Nachricht an Jarvis, nicht im Gruppen-Raum). Jarvis: Key in RAM-only Support-Session-Store ablegen, verknüpft mit session_token und expires_at. Key NIEMALS in DB persistieren. Zugriff auf Key nur möglich mit gültigem session_token UND actor ist in approved_agents Liste (Thomas hat approved_agents konfiguriert). Automatischer Key-Ablauf: Background-Job prüft alle 60 Sekunden ob expires_at überschritten — bei Ablauf: Key aus RAM löschen, Session status → expired. Revoke-Möglichkeit: Kunde kann jederzeit "ZUGANG WIDERRUFEN" schreiben → sofortiger Key-Löschung, Session status → revoked.
**Abnahme:** Key liegt nur im RAM, läuft automatisch ab, Revoke funktioniert sofort, kein Key in Logs oder DB.

#### SUPPORT-004: Support-Zugang im Admin Panel
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** ADMIN-002 + SUPPORT-003
**Beschreibung:**
Im Admin Panel Tab "Kunden" bei jedem Kunden: Support-Session Button. Wenn keine aktive Session: Button "Support-Session anfragen" (triggert SUPPORT-002 Flow). Wenn aktive Session: grüner Indikator "Session aktiv bis [ZEITSTEMPEL]", Button "Session anzeigen". Session-Ansicht zeigt: Qdrant Collection des Kunden (entschlüsselt für Dauer der Session), aktuelle Fehler-Logs des Kunden, MD Pool Inhalt (anonymisiert — echter Name bleibt [PERSON_A], aber Kontext lesbar), Aktionen-Log in Echtzeit. Welche Agents Zugriff haben: Thomas kann in Admin Panel Agents zur approved_agents Liste hinzufügen/entfernen (z.B. "Code Writer für diese Session freigeben"). Alle Klicks/Aktionen in Admin Panel → support_session_actions Tabelle.
**Abnahme:** Support-Zugang im Admin Panel sichtbar und nutzbar. Aktionen werden geloggt.

#### SUPPORT-005: Protokoll-Generierung nach Session-Ende
**Assignee:** Code Writer + ISDS Lead | **Priorität:** MITTEL | **Abhängigkeit:** SUPPORT-004
**Beschreibung:**
Nach Session-Ende (expired oder revoked): automatisch vollständiges Protokoll generieren. Protokoll enthält: Session-Übersicht (Mandant-ID, Dauer, wer hatte Zugriff, welche Agents), Chronologische Aktions-Liste aus support_session_actions (Timestamp, Actor, Action-Type, Detail), Bestätigung Key-Löschung (Timestamp), Fazit (was wurde gemacht, was wurde gelöst). Format: PDF via ReportLab oder WeasyPrint. Zwei Versionen: (1) Interne Version (vollständig, für audit_log) (2) Kunden-Version (gleicher Inhalt, aber ohne interne System-Details). Kunden-Version automatisch via Matrix an Kunden senden: "Dein Support-Session Protokoll ist angefügt." Interne Version in Nexus audit_log ablegen. Protokoll-Datei NICHT dauerhaft auf Server — nach Versand löschen.
**Abnahme:** Protokoll wird automatisch generiert, an Kunden gesendet, intern abgelegt. Datei danach gelöscht.

---

Aktualisiere auch die Nordstern-Architektur-Sektion im Workspace mit diesen zwei neuen Konzepten:

Füge in SYNTIQAI-NORDSTERN-2026-04-06.md nach Sektion 9 folgende neue Sektionen ein:

```
## 10. Admin Panel (SyntiqAI Backoffice)

Internes Web-Dashboard für Thomas (nur im Heimnetz, LXC 108 Port 8080).

Vier Bereiche:
- Kunden: alle Mandanten, Status, Kosten, Matrix-Link, Support-Session
- Templates: Anima-Konfigurationen, Deploy via Jarvis-Bestätigung
- System: LXC-Status, Agent-Status, Qdrant-Größen, API-Kosten
- Onboarding: Neukunden-Formular → Jarvis legt alles automatisch an

Onboarding-Prozess:
Thomas füllt Formular aus → Jarvis generiert mandant_id, Matrix-Raum,
Qdrant Collection, Anima-Template, Key → Key einmalig im Admin Panel
sichtbar → Thomas übergibt Key manuell an Kunden.

## 11. Support-Session (Break-Glass Access)

Temporärer Kunden-Key-Zugang für Wartung und Fehlersuche.

Prinzipien:
- Explizite Kunden-Einwilligung via Matrix (nicht implizit)
- Kunde wählt Dauer: 1h / 4h / 24h
- Key nur im RAM — niemals in DB oder Logs
- Automatischer Ablauf nach gewählter Dauer
- Sofortiger Widerruf durch Kunden jederzeit möglich
- Zugriff nur für Thomas + explizit freigegebene Agents
- Alle Aktionen in support_session_actions geloggt
- Vollständiges Protokoll an Kunden nach Session-Ende

DSGVO-Konformität:
- Rechtsgrundlage: explizite Einwilligung (Art. 6 Abs. 1 lit. a)
- Zeitliche Begrenzung + Protokoll = Rechenschaftspflicht (Art. 5 Abs. 2)
- Widerrufsmöglichkeit = Art. 7 Abs. 3 DSGVO
- Protokoll an Kunden = Transparenzpflicht (Art. 5 Abs. 1 lit. a)
```

---

### EPIC 8: KUNDEN-DASHBOARD
**Beschreibung:** Öffentlich erreichbares Self-Service Portal unter syntiq-ai.at/dashboard. Zeigt Nutzung, Guthaben/Abrechnung, Rechnungen, Support-Sessions. Unterstützt Prepaid und Postpaid.
**Label:** customer-portal, billing, public
**Assignee Epic:** Code Writer
**Abhängigkeit:** EPIC 3 (Nexus Billing) abgeschlossen + A1 Port-Problem gelöst

#### DASH-001: Kunden-Auth System
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** NEX-002
**Beschreibung:**
Magic Link Login via E-Mail (kein Passwort). Flow: E-Mail eingeben → einmaliger Link (15 Min) → Session 7 Tage. Alternativ: Matrix-Login (Kunde bestätigt Login in Matrix-Raum). Neue Nexus-Tabelle: customer_auth_tokens (id, mandant_id, email, token_hash, expires_at, used_at). E-Mail optional beim Onboarding — wenn nicht hinterlegt, nur Matrix-Login. Kein Passwort wird jemals gespeichert.
**Abnahme:** Magic Link + Matrix-Login funktionieren. Session 7 Tage gültig.

#### DASH-002: Dashboard Grundstruktur
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** DASH-001
**Beschreibung:**
Next.js unter syntiq-ai.at/dashboard (gleicher Vercel-Stack wie MKT-002). SyntiqAI Brand-Design (Navy + Cyan). Vier Bereiche: Übersicht, Guthaben/Abrechnung, Rechnungen, Support. Responsive (Mobile first). Logout + Session-Timeout-Warnung 5 Min vor Ablauf.
**Abnahme:** Dashboard erreichbar, Login funktioniert, vier Bereiche vorhanden, mobil nutzbar.

#### DASH-003: Nutzungsübersicht
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** DASH-002
**Beschreibung:**
Zeigt: Anfragen heute/Woche/Monat, verbrauchte API-Tokens (grob), aktive Services, letzter Anima-Kontakt. Liniendiagramm: Anfragen pro Tag letzte 30 Tage. Datenquelle: Nexus usage_logs nach mandant_id gefiltert. Wichtig: Nur aggregierte Daten — niemals Inhalte der Anfragen (Zero-Knowledge bleibt gewahrt).
**Abnahme:** Nutzungsdaten korrekt. Keine Inhalte sichtbar. Diagramm funktioniert.

#### DASH-004: Guthaben & Abrechnung (Prepaid + Postpaid)
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** DASH-003
**Beschreibung:**
Abrechnungsmodell in mandants Tabelle: billing_mode ENUM (prepaid/postpaid) — beim Onboarding gesetzt.

PREPAID: Guthaben in Euro, Verbrauch diese Woche, geschätzte Restlaufzeit, Farbindikator (grün >50% / gelb 20-50% / rot <20%), Button "Guthaben aufladen" (Phase 1: E-Mail an Thomas, Phase 2: Stripe). Automatische Matrix-Warnung bei <20%.

POSTPAID: Offener Betrag, nächstes Rechnungsdatum, Zahlungsstatus letzte Rechnung.

Neue Nexus-Tabellen (kein Breaking Change): mandant_balance (id, mandant_id, balance_eur, last_updated), balance_transactions (id, mandant_id, amount_eur, type, description, created_at).
**Abnahme:** Beide Modelle funktionieren. Prepaid-Warnung via Matrix aktiv. Aufladen-Button löst E-Mail an Thomas aus.

#### DASH-005: Rechnungsübersicht
**Assignee:** Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** DASH-004
**Beschreibung:**
Liste aller Rechnungen aus InvoiceNinja (via NEX-003 Sync) nach Mandant gefiltert. Spalten: Datum, Betrag, Status, PDF-Download. PDF direkt aus InvoiceNinja API. CSV-Export für Kundens Buchhaltung. Prepaid: zeigt balance_transactions History. Postpaid: zeigt InvoiceNinja Rechnungen.
**Abnahme:** Rechnungen sichtbar. PDF-Download + CSV-Export funktionieren.

#### DASH-006: Support-Bereich im Kunden-Dashboard
**Assignee:** Code Writer | **Priorität:** NIEDRIG | **Abhängigkeit:** DASH-002 + SUPPORT-002
**Beschreibung:**
Zeigt: aktive Support-Sessions mit Ablauf-Countdown + Widerrufs-Button, vergangene Sessions mit Protokoll-Download (PDF aus SUPPORT-005), Button "Support anfragen" (triggert SUPPORT-002 Flow), FAQ (statisch via Sanity CMS), Direktlink zum Matrix-Raum.
**Abnahme:** Sessions sichtbar, Widerruf funktioniert, Protokolle downloadbar, Support-Anfrage löst Matrix-Flow aus.

---

### EPIC 9: SCHLÜSSEL-MANAGEMENT & KEY-ESCROW
**Beschreibung:** Sicheres Backup-System für Kundenschlüssel via Shamir's Secret Sharing. Verhindert unwiederbringlichen Datenverlust bei Key-Verlust des Kunden. DSGVO-konform weil SyntiqAI alleine den Key nie rekonstruieren kann.
**Label:** security, key-management, dsgvo
**Assignee Epic:** ISDS Lead + Code Writer
**Abhängigkeit:** EPIC 4 (Enc-Agent) abgeschlossen

#### KEY-001: Shamir's Secret Sharing implementieren
**Assignee:** Code Writer | **Priorität:** HOCH | **Abhängigkeit:** ENC-002
**Beschreibung:**
Shamir's Secret Sharing (SSS) als Python-Modul. Library: secretsharing (pip). Schema: 2-von-3. Modul: /opt/syntiqai/lib/key_escrow.py.

Funktionen:
- split_key(key: bytes) -> tuple[str, str, str] — gibt (share1, share2, share3) als base58-Strings zurück
- reconstruct_key(share_a: str, share_b: str) -> bytes — rekonstruiert aus beliebigen 2 von 3 Shares

Wichtig: Nach Ausgabe der Shares alle digitalen Spuren sofort löschen (del shares, gc.collect()). Share 2 wird NIEMALS digital gespeichert — ausschliesslich physisch auf Papier.
**Abnahme:** reconstruct(share1, share2) == original_key. reconstruct(share2_alone) wirft ValueError. Kein Share in DB oder Logs nachweisbar.

#### KEY-002: Onboarding Key-Ausgabe Flow (Option A+C)
**Assignee:** Code Writer | **Priorität:** HOCH | **Abhängigkeit:** KEY-001 + ADMIN-005
**Beschreibung:**
Vollständiger 6-Schritte-Flow im Admin Panel. Exakte Reihenfolge:

SCHRITT 1 — Key + Shares generieren (automatisch, Jarvis):
Key = os.urandom(32). Via SSS in 3 Shares aufteilen. Alle Shares nur im RAM.

SCHRITT 2 — Share 1 digital an Kunden (automatisch, Jarvis):
Jarvis sendet Share 1 in einem PRIVATEN Matrix-Raum (nicht im Arbeits-Raum) an den Kunden: "Hier ist Ihr persönlicher Sicherheits-Share. Bitte sofort sicher speichern (z.B. Passwort-Manager): [SHARE_1_BASE58]". Nachricht wird nach 5 Minuten automatisch via Matrix Redaction API gelöscht.

SCHRITT 2b — Codewort vereinbaren (Thomas + Kunde, telefonisch/persönlich — KEIN digitaler Kanal):
Thomas vereinbart mit dem Kunden ein geheimes Codewort. Nicht auf den PDFs vermerkt. Im Admin Panel unter "Notfall-Codewort" eintragen — wird nur als bcrypt-Hash in mandants.emergency_code_hash gespeichert, niemals im Klartext. Zweck: Identitätsnachweis bei Notfall-Rekonstruktion (KEY-003).

SCHRITT 3 — Share 1 PDF für Kunden (Thomas druckt, übergibt physisch):
Admin Panel generiert PDF mit: Kundenname, mandant_id, Datum, QR-Code von Share 1, Share 1 als Klartext-String, Instruktionen auf Deutsch. Instruktions-Text: "Dies ist Ihr Sicherheits-Share für Ihre SyntiqAI-Daten. Bewahren Sie dieses Dokument sicher auf (z.B. Heimtresor). Bei Verlust kontaktieren Sie uns unter support@syntiq-ai.at." Thomas druckt aus und übergibt dem Kunden persönlich oder per Einschreiben.

SCHRITT 4 — Share 2 PDF für SyntiqAI-Tresor (Thomas druckt, versiegelt physisch):
Admin Panel generiert zweites PDF mit: mandant_id, Datum, QR-Code von Share 2, Share 2 als Klartext, Aufdruck "VERTRAULICH — NUR IM NOTFALL ÖFFNEN — GEMEINSAM MIT KUNDEN". Thomas: ausdrucken, in Umschlag versiegeln, Umschlag beschriften mit mandant_id + Datum, in Tresor einlagern (gemäss KEY-004).

SCHRITT 5 — Share 3 PDF als Backup für Kunden (Option C — Thomas druckt, übergibt physisch):
Admin Panel generiert drittes PDF mit Share 3, identisch zu Share 1 PDF aber mit Aufdruck: "BACKUP-KOPIE — Separat aufbewahren (anderer Ort als Hauptkopie, z.B. Bankschliessfach, Notar, Vertrauensperson)." Thomas übergibt dieses PDF ebenfalls dem Kunden (separat vom Share 1 PDF).

Was der Kunde am Ende hat:
- Share 1: digital selbst gespeichert + ausgedruckt (Heimort)
- Share 3: ausgedruckt (separater Ort)
- Rekonstruktion jederzeit selbst möglich: Share 1 + Share 3 = 2 von 3 = voller Key

Was SyntiqAI hat:
- Share 2: physisch versiegelt im Tresor
- Alleine NICHT rekonstruierbar (nur 1 von 3)
- Gemeinsam mit Kunde rekonstruierbar (Share 2 + Share 1 oder Share 3)

SCHRITT 6 — Bestätigung + digitale Spuren löschen:
Admin Panel zeigt Checkliste die Thomas manuell abhaken muss:
[ ] Share 1 digital via Matrix an Kunden gesendet
[ ] Share 1 PDF ausgedruckt und an Kunden übergeben/versendet
[ ] Share 2 PDF ausgedruckt, versiegelt, in Tresor eingelagert
[ ] Share 3 PDF ausgedruckt und an Kunden als Backup übergeben/versendet

Erst wenn alle vier gesetzt: Status wechselt von "onboarding" auf "aktiv". Danach: alle Share-Werte aus RAM, PDFs vom Server löschen. Audit-Log: "Key-Escrow abgeschlossen mandant_id [ID] — 3 Shares ausgegeben, keine digitale Kopie verblieben."

**Abnahme:** Alle 6 Schritte implementiert. PDFs korrekt generiert. Checkliste blockiert Aktivierung. Keine Share-Daten auf Server nach Abschluss nachweisbar.

#### KEY-003: Notfall-Rekonstruktions-Prozess
**Assignee:** ISDS Lead | **Priorität:** HOCH | **Abhängigkeit:** KEY-001
**Beschreibung:**
Notfall-Prozess dokumentieren und als Admin-Panel-Flow implementieren. Ablauf wenn Kunde Share 1 verloren hat: (1) Kunde kontaktiert SyntiqAI via Matrix oder E-Mail. (2) Identitätsprüfung: Video-Call mit Thomas, Lichtbildausweis zeigen, Codewort aus Onboarding-Dokument nennen. (3) Thomas öffnet physischen Umschlag (Share 2) im Beisein des Kunden (Video-Call). (4) Gemeinsam in Admin Panel: Share 2 eingeben + Kunde gibt Share 1 ein (falls noch irgendwo vorhanden) oder Share 3 (Treuhänder) → Key rekonstruiert. (5) Neuen Key generieren, alte Qdrant Collection re-verschlüsseln, neue Shares ausgeben, alten Umschlag vernichten. (6) Gesamter Prozess wird in audit_log dokumentiert. Prozess-Dokument: /home/claw/.openclaw/workspace/KEY-003-Notfall-Prozess.md
**Abnahme:** Prozess dokumentiert. Admin Panel hat "Notfall-Rekonstruktion" Button (nur für Thomas sichtbar). Audit-Log wird geschrieben.

#### KEY-004: SYN — Thomas entscheidet: Tresor-Lösung
**Assignee:** Thomas | **Priorität:** HOCH | **Thomas-Entscheidung zwingend**
**Beschreibung:**
Wo werden die physischen Share-2-Umschläge verwahrt? Optionen: (A) Privater Tresor zu Hause (günstig, aber bei Hausbrand verloren). (B) Bankschließfach (sicher, aber Zugriff dauert). (C) Feuerfester Dokumententresor (Kompromiss). Thomas entscheidet und dokumentiert die gewählte Lösung. Diese Information fließt ins Kunden-Onboarding-Dokument ein ("Ihre Sicherheitskopie wird in [LÖSUNG] aufbewahrt").
**Abnahme:** Thomas hat entschieden und Entscheidung dokumentiert. Onboarding-Vorlage aktualisiert.

---

### EPIC 10: MONITORING & BETRIEB
**Beschreibung:** Produktions-Monitoring, Alerting, Rate-Limiting, Log-Management. Muss vor erstem produktivem Kunden laufen.
**Label:** monitoring, operations, infrastructure
**Assignee Epic:** DevOps Agent
**Abhängigkeit:** EPIC 2 abgeschlossen

#### MON-001: Uptime Kuma deployen
**Assignee:** DevOps Agent | **Priorität:** KRITISCH | **Abhängigkeit:** ENC-001 (LXC 111 existiert)
**Beschreibung:**
Uptime Kuma (self-hosted Monitoring) via Docker auf LXC 111 neben Qdrant deployen (Port 3001). Monitors anlegen für: matrix (.104, HTTP-Check Port 8448), nexus-billing (.108, Port 8080), invoiceninja-test (.110, Port 80), qdrant-enc (.111, Port 6333), customer-01 (.150, Port 80), openclaw (.96, Port 18789), nginxproxy (.105, Port 80). Check-Intervall: 60 Sekunden. Alert bei Ausfall: Matrix-Nachricht an Thomas (Uptime Kuma hat Matrix-Notification-Integration). Alert-Schwelle: nach 2 fehlgeschlagenen Checks (nicht sofort — verhindert False Positives). Statusseite: intern erreichbar unter 192.168.178.111:3001. Später: öffentliche Statusseite status.syntiq-ai.at (nach A1-Port-Lösung).
**Abnahme:** Alle Services überwacht. Alert via Matrix bei simuliertem Ausfall funktioniert. Dashboard zeigt Uptime-History.

#### MON-002: Rate-Limiting pro Mandant
**Assignee:** Code Writer | **Priorität:** KRITISCH | **Abhängigkeit:** PILOT-003 (Matrix Bot)
**Beschreibung:**
Rate-Limiting im Matrix-Bot und KAM-Layer implementieren. Limits pro Mandant: max 60 Anfragen/Stunde, max 500 Anfragen/Tag, Hard-Stop bei Prepaid-Guthaben = 0. Implementierung: Redis auf LXC 111 (Docker, neben Qdrant) als Counter-Store. Bei Überschreitung: Anima antwortet dem Kunden: "Du hast dein Stunden-Limit erreicht. Bitte warte 60 Minuten oder kontaktiere uns für eine Anpassung." Bei Prepaid = 0: "Dein Guthaben ist aufgebraucht. Bitte lade Guthaben auf um weiterzumachen." Thomas bekommt Matrix-Alert wenn ein Kunde 80% seines Tageslimits erreicht. Limits sind pro Mandant in mandants Tabelle konfigurierbar (neue Felder: rate_limit_hourly, rate_limit_daily — kein Breaking Change).
**Abnahme:** Rate-Limit greift nach 60 Anfragen/h. Prepaid-Stop bei 0 Guthaben. Thomas-Alert bei 80%.

#### MON-003: Log-Retention Policy implementieren
**Assignee:** DevOps Agent + ISDS Lead | **Priorität:** MITTEL
**Beschreibung:**
Log-Typen und Aufbewahrungsfristen definieren und technisch umsetzen:
System-Logs (LXC, Docker): 30 Tage → automatisch rotieren (logrotate)
Paperclip Agent-Logs: 14 Tage → automatisch löschen
Nexus audit_log: 3 Jahre → DSGVO-Pflicht für Buchführung
support_session_actions: 1 Jahr → nach Ablauf löschen
Uptime Kuma History: 90 Tage
Matrix-Bot Logs: 7 Tage (keine Inhalte, nur Timestamps + Status)
Cron-Job täglich 04:00: bereinigt alle Logs nach Policy. Dokumentation: /home/claw/.openclaw/workspace/MON-003-Log-Retention-Policy.md
**Abnahme:** Cron-Job läuft. Logs werden nach Policy bereinigt. Dokumentation ins VVT eingetragen.

#### MON-004: Disaster Recovery Dokument
**Assignee:** ISDS Lead | **Priorität:** MITTEL | **Thomas-Review zwingend**
**Beschreibung:**
Disaster Recovery Plan erstellen. Definiert: RTO (Recovery Time Objective — wie lange darf der Ausfall dauern?), RPO (Recovery Point Objective — wie viel Datenverlust ist tolerierbar?). Szenarien: (1) Einzelner LXC fällt aus → Restore aus Proxmox Backup (Ziel: <2h). (2) Proxmox Host fällt aus → Restore auf neuem Host aus Synology-Backup (Ziel: <24h). (3) Synology NAS fällt aus → kein Backup-Zugriff, nur Live-System (Risiko dokumentieren). (4) Komplettausfall (Brand, etc.) → alle Kundendaten verloren wenn Shares verloren → Key-Escrow-Prozess (KEY-003) greift. Recovery-Schritte pro Szenario dokumentieren. Thomas muss RTO/RPO genehmigen — diese Werte fließen in Kunden-SLA ein.
**Abnahme:** Dokument erstellt, Thomas genehmigt RTO/RPO, Werte in SLA-Vorlage eingetragen.

#### MON-005: Offboarding-Prozess implementieren
**Assignee:** Code Writer + ISDS Lead | **Priorität:** MITTEL | **Abhängigkeit:** ADMIN-005
**Beschreibung:**
Gegenstück zum Onboarding. "Kunden kündigen" Button im Admin Panel. Offboarding-Checkliste (Jarvis führt automatisch aus, Thomas bestätigt jeden Schritt): (1) Anima-Zugang sperren (Matrix-Bot antwortet nicht mehr). (2) Rate-Limits auf 0 setzen. (3) Kundendaten-Löschfrist starten (30 Tage Aufbewahrung nach DSGVO Art. 17, dann automatisch löschen). (4) InvoiceNinja Kunde deaktivieren. (5) Offene Rechnungen prüfen und Thomas melden. (6) Qdrant Collection nach 30 Tagen löschen (geplanter Job). (7) Key-Escrow-Umschlag (Share 2) vernichten — Thomas bestätigt physisch. (8) Kunden-Löschbestätigung senden (DSGVO Pflicht): Matrix-Nachricht + E-Mail mit Datum der vollständigen Löschung. (9) Eintrag in audit_log. Neue Nexus-Tabelle: offboarding_jobs (mandant_id, status, scheduled_deletion_date, steps_completed JSON).
**Abnahme:** Kompletter Offboarding-Flow durchgespielt mit Test-Mandant. Löschbestätigung wird gesendet. Audit-Log korrekt.

---

### EPIC 11: RECHTLICHES & COMPLIANCE
**Beschreibung:** Fehlende rechtliche Dokumente und Compliance-Anforderungen. Mehrere Tasks erfordern Thomas-Entscheidung.
**Label:** legal, compliance, governance
**Assignee Epic:** ISDS Lead

#### LEGAL-001: Servicevertrag + AGB Vorlage
**Assignee:** ISDS Lead | **Priorität:** HOCH | **Thomas-Review zwingend**
**Beschreibung:**
Vorlage für Servicevertrag erstellen. Enthält: Leistungsbeschreibung (was leistet SyntiqAI, was nicht), Preismodell-Platzhalter (Thomas füllt aus), Laufzeit und Kündigungsfristen (Empfehlung: monatlich kündbar mit 30 Tagen Frist), SLA-Verweis (aus MON-004), Haftungsbeschränkung (KRITISCH: Haftungsausschluss für KI-Fehler in Angeboten/Rechnungen — Anima macht Vorschläge, Kunde trägt Verantwortung für finale Freigabe), Datenschutz-Verweis (AVV als Anlage). Separate AGB-Vorlage: Zahlungsbedingungen, Prepaid/Postpaid, Mahnwesen, Datenschutz-Kurzfassung. Alles zur juristischen Prüfung vorbereiten — zusammen mit DSGVO-Bewertungsunterlage zum Juristen. Workspace: /home/claw/.openclaw/workspace/LEGAL-001-Servicevertrag-AGB-Vorlage.md
**Abnahme:** Vorlagen erstellt. Thomas reviewed. Zum Juristen weitergeleitet.

#### LEGAL-002: SYN — Thomas: Preismodell festlegen
**Assignee:** Thomas | **Priorität:** HOCH | **Thomas-Entscheidung zwingend**
**Beschreibung:**
Preismodell für SyntiqAI definieren. Zu entscheiden: (A) Grundgebühr/Monat pro Kunde (Empfehlung: €49-149/Monat je nach Plan). (B) Verbrauchsbasierte Komponente (pro API-Call oder pro Token — Empfehlung: pro 1000 Tokens, ca. €0.02-0.05). (C) Prepaid-Pakete (z.B. €50 / €100 / €200). (D) Postpaid-Limit (maximale Monatsrechnung ohne vorherige Genehmigung). (E) Preise für Add-ons (InvoiceNinja-Hosting: +€20/Monat). Thomas entscheidet, trägt Preise in Servicevertrag-Vorlage ein, Nexus-Billing wird entsprechend konfiguriert.
**Abnahme:** Preismodell dokumentiert und von Thomas freigegeben.

#### LEGAL-003: SYN — Thomas: Gewerbeanmeldung prüfen
**Assignee:** Thomas | **Priorität:** HOCH | **Thomas-Entscheidung zwingend**
**Beschreibung:**
Für echte Kundenabrechnung (auch mit Flo) braucht SyntiqAI eine angemeldete Rechtsform. Optionen: (A) Einzelunternehmen — sofort möglich, WKO-Anmeldung, günstiger Start. (B) GmbH — höhere Kosten (~€500 Gründungsgebühr + Notar), aber Haftungsbeschränkung. Empfehlung: Einzelunternehmen für Pilot, GmbH wenn >3 zahlende Kunden. Thomas prüft und entscheidet. Falls Einzelunternehmen: WKO-Anmeldung als IT-Dienstleister (NACE 6201/6209). UID-Nummer beantragen (Finanzamt). Impressum für Website damit befüllen.
**Abnahme:** Entscheidung getroffen, Anmeldung eingeleitet oder bereits vorhanden.

#### LEGAL-004: SYN — Thomas: EU AI Act Bewertung beauftragen
**Assignee:** Thomas | **Priorität:** MITTEL | **Thomas-Entscheidung zwingend**
**Beschreibung:**
SyntiqAI verarbeitet Geschäftskommunikation und unterstützt Entscheidungen (Angebote, Rechnungen) für KMUs. Mögliche Klassifizierung nach EU AI Act: "Limited Risk" (Transparenzpflicht — Nutzer muss wissen er interagiert mit KI) oder "High Risk" (wenn KI Entscheidungen trifft die rechtliche/wirtschaftliche Konsequenzen haben). Thomas beauftragt Juristen mit einer kurzen EU AI Act Einschätzung — gleichzeitig mit DSGVO-Bewertung. Sofort-Maßnahme die unabhängig davon gilt: Anima muss sich beim ersten Kontakt als KI zu erkennen geben ("Ich bin Anima, ein KI-Assistent von SyntiqAI.").
**Abnahme:** Jurist wurde beauftragt. Anima-Prompt enthält KI-Identifikation.

#### LEGAL-005: Impressum & Website-Pflichtangaben
**Assignee:** ISDS Lead | **Priorität:** HOCH | **Abhängigkeit:** LEGAL-003
**Beschreibung:**
Österreichisches Pflichtimpressum nach §5 ECG für syntiq-ai.at erstellen. Enthält: vollständiger Name (Thomas Z.), Unternehmensbezeichnung (SyntiqAI), Adresse (Baden bei Wien), E-Mail-Kontakt, UID-Nummer (nach LEGAL-003), Gewerbebehörde (Magistrat/BH Baden), Berufsbezeichnung (IT-Dienstleister), anwendbare Rechtsvorschriften (GewO). Als Sanity CMS Inhalt vorbereiten (für MKT-002). Datenschutz-Link auf syntiq-ai.at/datenschutz. Workspace: /home/claw/.openclaw/workspace/LEGAL-005-Impressum.md — Thomas füllt fehlende Daten ein.
**Abnahme:** Impressum vollständig (nach LEGAL-003 Entscheidung), Thomas freigegeben, auf Website publiziert.

#### LEGAL-005b: SYN — Thomas: A1 Port-Problem lösen
**Assignee:** Thomas | **Priorität:** HOCH | **Thomas-Entscheidung zwingend**
**Beschreibung:**
Ohne öffentlich erreichbare IP sind Kunden-Dashboard (syntiq-ai.at/dashboard) und öffentliche Website nicht erreichbar. Aktuelle Situation: feste IP vorhanden (A1 Business) aber Ports kommen nicht durch (vermutlich Zyxel NR7101 5G + FritzBox Double-NAT Problem). Zu tun: (1) A1 Support kontaktieren, IP-Passthrough oder Bridge-Mode für Zyxel anfragen. Gesprächsleitfaden: "Ich habe einen A1 Business Anschluss mit fester IP. Mein Zyxel NR7101 steht vor meiner FritzBox. Ich brauche entweder Bridge-Mode am Zyxel oder IP-Passthrough damit meine FritzBox die echte öffentliche IP bekommt und ich Ports freigeben kann." (2) Nach Lösung: Nginx Proxy Manager (.105) konfigurieren für syntiq-ai.at → LXC 300 (InvoiceNinja) und syntiq-ai.at/dashboard → Kunden-Dashboard. (3) DNS A-Record auf feste IP setzen. Thomas dokumentiert Lösung im Workspace.
**Abnahme:** curl https://syntiq-ai.at gibt HTTP-Response. Kunden-Dashboard unter syntiq-ai.at/dashboard erreichbar.

#### LEGAL-006: SLA-Dokument erstellen
**Assignee:** ISDS Lead | **Priorität:** MITTEL | **Thomas-Review zwingend**
**Beschreibung:**
Service Level Agreement (SLA) Vorlage erstellen. Basiert auf RTO/RPO aus MON-004. Definiert: Verfügbarkeit (Empfehlung: 99% = max 7h Ausfall/Monat — realistisch für Homelab), Wartungsfenster (z.B. Sonntag 02:00-06:00 ohne Ankündigung), Response-Zeit bei Supportanfragen (Empfehlung: <4h während Geschäftszeiten Mo-Fr 08-18), Eskalationspfad, Ausnahmen (höhere Gewalt, geplante Wartung). Thomas muss SLA-Werte genehmigen — er muss sie auch einhalten können. Workspace: /home/claw/.openclaw/workspace/LEGAL-006-SLA-Vorlage.md
**Abnahme:** SLA erstellt, Thomas genehmigt Werte, in Servicevertrag eingebunden.

---

### EPIC 12: DEVELOPER EXPERIENCE & BYOA
**Beschreibung:** API-Dokumentation und Onboarding für BYOA-Kunden (Bring Your Own Agent). Ohne das ist Modell B nicht verkaufbar.
**Label:** api, byoa, documentation
**Assignee Epic:** Documentation Agent
**Abhängigkeit:** EPIC 3 abgeschlossen

#### DEV-001: GitHub Repository Struktur anlegen
**Assignee:** DevOps Agent | **Priorität:** MITTEL
**Beschreibung:**
Klare Repository-Struktur in syntiqAI GitHub Organisation anlegen. Repositories: syntiqAI/paperclip (Fork — existiert), syntiqAI/nexus-billing (neues Repo), syntiqAI/enc-agent (neues Repo), syntiqAI/admin-panel (neues Repo), syntiqAI/customer-dashboard (neues Repo), syntiqAI/anima-templates (neues Repo — enthält YAML/JSON Template-Dateien), syntiqAI/syntiq-ai-website (für MKT-002). Jedes Repo: README.md mit Beschreibung, .gitignore, LICENSE (proprietär außer paperclip-Fork). Branch-Strategie: main (produktiv), develop (integration), feature/* (feature branches). Kein Code wird direkt in main gepusht — immer via PR. Jarvis erstellt PRs, Thomas reviewed und mergt (oder Jarvis bei Non-Breaking Changes autonom).
**Abnahme:** Alle Repos angelegt, README vorhanden, Branch-Strategie dokumentiert.

#### DEV-002: BYOA API Spezifikation
**Assignee:** Documentation Agent + Code Writer | **Priorität:** MITTEL | **Abhängigkeit:** DEV-001
**Beschreibung:**
OpenAPI 3.0 Spezifikation für die SyntiqAI BYOA API erstellen. Endpoints die ein externer Agent braucht: POST /api/v1/agent/message (Nachricht einreichen, mit mandant_id + session_token), GET /api/v1/agent/skills (verfügbare Skills für diesen Mandanten), GET /api/v1/agent/context (aktueller Kontext aus MD Pool, anonymisiert), POST /api/v1/agent/feedback (Qualitäts-Feedback). Auth: API-Key im Header (X-SyntiqAI-Key). Rate-Limiting: gleich wie für Anima. Swagger UI unter 192.168.178.108:8080/api-docs (intern). Später: öffentlich unter syntiq-ai.at/developers. YAML-Datei in syntiqAI/nexus-billing Repo.
**Abnahme:** OpenAPI Spec vollständig, Swagger UI erreichbar, alle Endpoints implementiert und dokumentiert.

---

### EPIC 13: TEST-INFRASTRUKTUR
**Beschreibung:** Formeller Test-Mandant und Test-Prozesse bevor Änderungen auf Produktiv-Kunden ausgerollt werden.
**Label:** testing, quality
**Assignee Epic:** Code Writer
**Abhängigkeit:** EPIC 4 abgeschlossen

#### TEST-001: Test-Mandant anlegen
**Assignee:** DevOps Agent | **Priorität:** MITTEL | **Abhängigkeit:** PILOT-005
**Beschreibung:**
Formellen Test-Mandanten anlegen: mandant_id: "syntiqai-internal-test", Name: "SyntiqAI Testumgebung". Eigene Qdrant Collection, eigener Matrix-Raum (@test:syntiq-ai.at), eigener InvoiceNinja-Testkunde, eigener Key (in Workspace gespeichert — kein echtes Zero-Knowledge hier, das ist intern). Jede Änderung an Anima-Templates, KAM-Konfiguration, oder Enc-Agent wird ZUERST auf Test-Mandant getestet. Test-Protokoll: Checkliste im Workspace die Jarvis nach jedem Test ausfüllt. Erst nach erfolgreichem Test: Ausrollen auf Produktiv-Mandanten.
**Abnahme:** Test-Mandant aktiv. Erster vollständiger Flow-Test durchgelaufen und dokumentiert.

---

### BACKLOG (Thomas zugewiesen — keine Tasks in Paperclip, nur Dokumentation)

Folgende Punkte werden als Backlog-Dokument im Workspace abgelegt:
`/home/claw/.openclaw/workspace/SYNTIQAI-BACKLOG.md`

Inhalt des Backlog-Dokuments:

```
# SyntiqAI Backlog

## Phase 2 Features (nach ersten 3 zahlenden Kunden)
- Stripe Integration für Prepaid-Guthaben aufladen
- Öffentliche Statusseite status.syntiq-ai.at
- Multi-Language Support für Anima (auto-detect)
- Changelog-System: "Was ist neu" monatlich via Matrix
- Datenmigrations-Strategie (falls Qdrant gewechselt wird)
- WCAG 2.1 AA Accessibility Audit für Kunden-Dashboard
- Lokales KAM-Hosting wenn GPU-Hardware verfügbar

## Phase 3 Features (Skalierung)
- GmbH-Gründung wenn >3 zahlende Kunden
- Managed Backups für Kunden-Keys (Hardware Security Module)
- Multi-Region Hosting (aktuell nur Baden bei Wien)
- Reseller-Modell (IT-Dienstleister wie Flo verkaufen SyntiqAI weiter)
- White-Label Option (Kunde nutzt eigene Domain + Branding)

## Offene technische Schulden
- Disaster Recovery testen (nicht nur dokumentieren)
- Penetration Test der gesamten Infrastruktur
- Dependency-Audit aller Python/Node-Libraries
```

---

Aktualisiere auch SYNTIQAI-NORDSTERN-2026-04-06.md mit diesen neuen Sektionen am Ende:

```
## 13. Schlüssel-Management (Key-Escrow)

Shamir's Secret Sharing (2-von-3 Schema):
- Share 1: Kunde (digital + ausgedruckt als QR-Code)
- Share 2: SyntiqAI physisch (versiegelter Umschlag, Tresor)
- Share 3: optional Treuhänder des Kunden (Notar/Steuerberater)

SyntiqAI alleine kann Key NIE rekonstruieren (Zero-Knowledge bleibt gewahrt).
Notfall: Kunde weist sich aus (Video-Call + Lichtbildausweis + Codewort)
→ gemeinsame Rekonstruktion aus Share 1 + Share 2.
Gesamter Notfall-Prozess wird in audit_log dokumentiert.

## 14. Monitoring & Betrieb

Uptime Kuma: alle LXCs überwacht, Alert via Matrix bei Ausfall.
Rate-Limiting: 60 Anfragen/h, 500/Tag pro Mandant.
Hard-Stop: Prepaid-Guthaben = 0 → kein weiterer Zugriff.
Log-Retention: System 30d, Agent 14d, Audit 3 Jahre, Session 1 Jahr.
RTO/RPO: definiert in MON-004, von Thomas genehmigt.

## 15. Offboarding

30 Tage nach Kündigung: alle Kundendaten gelöscht.
Schritte: Zugang sperren → Rechnungen klären → 30d warten
→ Qdrant löschen → Key-Escrow vernichten → Bestätigung senden.
Alle Schritte in audit_log + Löschbestätigung an Kunden (DSGVO Art. 17).

## 16. Rechtliches (Checkliste)

Vor Produktivbetrieb zwingend:
- Gewerbeanmeldung (LEGAL-003)
- Servicevertrag + AGB (LEGAL-001)
- Impressum (LEGAL-005)
- SLA (LEGAL-006)
- Preismodell (LEGAL-002)

Parallel zum Juristen:
- DSGVO-Bewertungsunterlage v1.2 (liegt vor)
- EU AI Act Einschätzung (LEGAL-004)
- Anima identifiziert sich immer als KI beim ersten Kontakt
```

---

1. **P1 — Kein Hard-Coding von Modellen:** Immer get_model() verwenden — nie Modellnamen direkt im Code
2. **P3 — Breaking Changes:** Tabelle aus Nordstern Sektion 9 beachten — Thomas fragen bevor Breaking Change
3. **Kein Deployment auf LXC 300** bevor DSGVO-001 + DSGVO-002 von Thomas freigegeben sind
4. **Kein Telegram-Bot implementieren** bevor DSGVO-007 freigegeben ist
5. **IN_PROGRESS nur setzen** wenn tatsächlich aktiv gearbeitet wird
6. **Alle Credentials** in ENV-Variablen oder OpenClaw Config — niemals in Dateien oder Workspace
7. **Keys niemals loggen** — weder Kunden-Keys noch API-Keys noch Support-Session-Keys im Klartext
8. **Support-Session Keys:** Ausschließlich RAM, niemals DB, niemals Logs, automatischer Ablauf, bei Ablauf sofort gc.collect()
9. **Thomas nur kontaktieren** wenn: technisch blockiert, Breaking Change geplant, echte Unklarheit die nicht aus Nordstern auflösbar

---

*Human Principal: Thomas Z. | Nordstern v1.9 | 2026-04-06 | SyntiqAI*
