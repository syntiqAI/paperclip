# SKILL: invoiceninja_billing

## Metadaten
- **Skill-ID:** invoiceninja_billing
- **Version:** 1.0.0
- **Autor:** SyntiqAI / Jarvis
- **Trigger-Keywords:** rechnung, invoice, kunde, client, zahlung, payment, mahnung, billing, faktura, offene posten, überfällig

---

## Beschreibung
Dieser Skill gibt Jarvis die Fähigkeit, das interne SyntiqAI InvoiceNinja-System 
vollständig zu steuern. Jarvis kann Kunden anlegen, Rechnungen erstellen und 
versenden, Zahlungen erfassen und offene Posten überwachen.

---

## API-Konfiguration

```
BASE_URL = http://192.168.178.150:9000/api/v1
HEADERS:
  X-API-TOKEN: DEIN_TOKEN_HIER
  Content-Type: application/json
  X-Requested-With: XMLHttpRequest
```

---

## Fähigkeiten & Endpoints

### 1. Kunden (Clients)

#### Alle Kunden auflisten
```
GET /clients?per_page=50
```

#### Kunden suchen
```
GET /clients?filter={suchbegriff}
```

#### Kunden anlegen
```
POST /clients
{
  "name": "Firmenname",
  "contacts": [{
    "first_name": "Vorname",
    "last_name": "Nachname",
    "email": "email@example.com",
    "phone": "+43..."
  }],
  "address1": "Straße",
  "city": "Stadt",
  "country_id": "40"
}
```
> Hinweis: country_id 40 = Österreich

#### Kunden-Detail abrufen
```
GET /clients/{client_id}
```

---

### 2. Rechnungen (Invoices)

#### Alle Rechnungen auflisten
```
GET /invoices?per_page=50
```

#### Offene/überfällige Rechnungen
```
GET /invoices?filter=overdue
GET /invoices?status=unpaid
```

#### Rechnung erstellen
```
POST /invoices
{
  "client_id": "{client_id}",
  "date": "YYYY-MM-DD",
  "due_date": "YYYY-MM-DD",
  "line_items": [
    {
      "product_key": "Leistungsbezeichnung",
      "notes": "Detailbeschreibung",
      "cost": 100.00,
      "quantity": 1,
      "tax_name1": "USt",
      "tax_rate1": 20
    }
  ],
  "public_notes": "Zahlbar innerhalb von 14 Tagen.",
  "terms": "Netto 14 Tage"
}
```

#### Rechnung versenden (E-Mail)
```
POST /invoices/{invoice_id}/email
```

#### Rechnung als PDF-Link
```
GET /invoices/{invoice_id}/download
```

#### Rechnungsstatus ändern
```
PUT /invoices/{invoice_id}
{ "status_id": "{status}" }
```
Status-Codes:
- 1 = Draft (Entwurf)
- 2 = Sent (Versendet)
- 3 = Partial (Teilbezahlt)
- 4 = Paid (Bezahlt)
- 5 = Cancelled (Storniert)

#### Mahnung senden
```
POST /invoices/{invoice_id}/email
{
  "template": "reminder1"
}
```
Templates: reminder1, reminder2, reminder3

---

### 3. Zahlungen (Payments)

#### Zahlungen auflisten
```
GET /payments?per_page=50
```

#### Zahlung erfassen
```
POST /payments
{
  "client_id": "{client_id}",
  "date": "YYYY-MM-DD",
  "amount": 100.00,
  "invoices": [
    {
      "invoice_id": "{invoice_id}",
      "amount": 100.00
    }
  ],
  "type_id": "1"
}
```
Zahlungsarten (type_id):
- 1 = Banküberweisung
- 2 = Cash
- 4 = Kreditkarte

---

### 4. Produkte (Products)

#### Produkte auflisten
```
GET /products
```

#### Produkt anlegen
```
POST /products
{
  "product_key": "CONSULTING",
  "notes": "IT-Beratung / Consulting",
  "cost": 120.00,
  "tax_name1": "USt",
  "tax_rate1": 20
}
```

---

## Verhaltensregeln (PFLICHT)

1. **Bestätigung vor Aktionen:** Vor dem Erstellen/Versenden einer Rechnung 
   immer eine Zusammenfassung zeigen und explizite Bestätigung einholen.

2. **Niemals ohne Verifizierung:** Kundennamen immer via API verifizieren 
   (GET /clients?filter=...) bevor eine Rechnung erstellt wird. Nie aus 
   dem Gedächtnis nehmen.

3. **Destruktive Aktionen:** Stornierungen und Löschungen immer mit 
   expliziter Warnung und doppelter Bestätigung absichern.

4. **Beträge:** Immer in EUR mit 2 Dezimalstellen ausgeben. 
   Österreichische USt (20%) standardmäßig aufschlagen sofern nicht 
   anders angegeben.

5. **Fehlerbehandlung:** API-Fehler klar auf Deutsch erklären, 
   HTTP-Statuscode nennen und konkreten Lösungsvorschlag machen.

6. **Datenschutz:** Keine Kundendaten in Logs oder Antworten 
   unnötig exponieren.

---

## Beispiel-Workflows

### Workflow A: Neue Rechnung für bestehenden Kunden
```
1. GET /clients?filter={kundenname}          → Client-ID ermitteln
2. Zusammenfassung zeigen (Kunde, Betrag, Leistung, Fälligkeit)
3. Bestätigung abwarten
4. POST /invoices                            → Rechnung erstellen
5. POST /invoices/{id}/email                 → Rechnung versenden
6. Bestätigung ausgeben mit Invoice-Nummer
```

### Workflow B: Offene Posten Check
```
1. GET /invoices?filter=overdue              → Überfällige laden
2. Sortiert nach Fälligkeitsdatum ausgeben
3. Gesamtbetrag der offenen Forderungen nennen
4. Optional: Mahnungen auf Anfrage versenden
```

### Workflow C: Neuen Kunden + erste Rechnung
```
1. POST /clients                             → Kunde anlegen
2. Client-ID aus Response nehmen
3. Zusammenfassung der Kundendaten bestätigen lassen
4. POST /invoices mit neuer Client-ID        → Rechnung erstellen
5. Optional sofort versenden
```

### Workflow D: Zahlung erfassen
```
1. GET /clients?filter={name}               → Client-ID
2. GET /invoices?client_id={id}&status=unpaid → Offene Rechnungen
3. Zusammenfassung zeigen
4. POST /payments                           → Zahlung buchen
5. Status der Rechnung prüfen (sollte auf Paid springen)
```

---

## Ausgabeformat

Bei Rechnungsübersichten immer tabellarisch:
```
| # | Kunde | Betrag | Fällig | Status |
|---|-------|--------|--------|--------|
| 0001 | Mustermann GmbH | €500,00 | 2026-04-21 | Offen |
```

Bei Einzelrechnungen vollständige Details mit allen Positionen.

---

## Skill laden
Dieser Skill wird automatisch aktiviert wenn der User Begriffe wie 
"Rechnung", "Invoice", "Kunde anlegen", "Zahlung", "Mahnung", 
"offene Posten" oder "Billing" verwendet.
