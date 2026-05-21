# FitnessAbrechnung – Aufgabenliste & Dokumentation
**Projekt:** Fitti Exempel GmbH · Auftragnehmer: JCR IT Solution GmbH  
**Pflichtenheft:** Version 1.2 · **Frontend-Meilenstein M3:** bis 27.05.2026

---

## Was wurde bereits erstellt (HTML + CSS)

| Datei | Inhalt | Status |
|---|---|---|
| `src/assets/main.css` | Globales Design-System (Dark Theme, Tokens, Utilities) | ✅ Fertig |
| `src/App.vue` | App-Shell mit Sidebar-Navigation, Routing per `activeView` | ✅ Fertig |
| `src/components/KundenVerwaltung.vue` | Kundenliste, Filter, Modal-Formular | ✅ Fertig |
| `src/components/AboVerwaltung.vue` | Abo-Karten-Grid, Tabelle, Modal-Formular | ✅ Fertig |
| `src/components/AbrechnungsUebersicht.vue` | Monats-/Jahresübersicht, Balkendiagramm, Kundenabrechnung | ✅ Fertig |
| `src/components/EinstellungenView.vue` | Systeminfos, API-Status-Konfiguration | ✅ Fertig |

---

## Deine Aufgaben – Vue-Logik (JavaScript)

### App.vue
- [ ] `data()` mit initialen Testdaten für `kunden[]`, `abos[]`, `ermaessigungen[]` befüllen
- [ ] `mounted()` – API-Calls für Initialdaten (GET /api/kunden, GET /api/abos)

---

### KundenVerwaltung.vue
- [ ] `speichereKunde()` – API POST (Neuanlage) oder PUT (Bearbeiten) aufrufen
- [ ] `loescheKunde(nr)` – API DELETE aufrufen
- [ ] Nach API-Response: `$emit('kunden-updated', neueKunden)` auslösen
- [ ] Validierung: IBAN-Format prüfen (mind. 15 Zeichen)
- [ ] **Testfall T1** aus Pflichtenheft: Kunden anlegen → Abo zuweisen → Abrechnung prüfen

---

### AboVerwaltung.vue
- [ ] `speichereAbo()` – API POST (Neuanlage) oder PUT (Bearbeiten) aufrufen
- [ ] `loescheAbo(nr)` – API DELETE aufrufen
- [ ] Nach API-Response: `$emit('abos-updated', neueAbos)` auslösen
- [ ] **Testfall T2** aus Pflichtenheft: Grundpreis ändern → persistent gespeichert

---

### AbrechnungsUebersicht.vue
- [ ] `ladeMonat()` – GET /api/abrechnungen?monat=YYYY-MM → `monatsAbrechnungen[]` befüllen
- [ ] `ladeJahr()` – GET /api/abrechnungen?jahr=YYYY → `jahresMonatsDaten[]` befüllen
- [ ] `ladeKundenabrechnung()` – GET /api/abrechnungen?kundenNr=&zeitraum=
- [ ] `exportCSV()` – Daten als CSV-Datei herunterladen (Browser-Download)
- [ ] `exportPDF()` – z.B. mit `window.print()` oder PDF-Bibliothek
- [ ] **Testfall T3** aus Pflichtenheft: Monat laden → Summe stimmt → Export ausführen

---

### EinstellungenView.vue
- [ ] `testApiVerbindung()` – GET /api/health → `apiStatus` setzen (bereits als Stub vorhanden)

---

## Pflichtenheft-Kriterien – Abdeckung

### ✅ Muss-Kriterien
| Kriterium | Abgedeckt in |
|---|---|
| Abrechnung für jeden Kunden | KundenVerwaltung + AbrechnungsUebersicht (Tab "Kundenabrechnung") |
| Ermäßigte Abos | KundenVerwaltung (Feld `ermaessigungssatz`), Preisvorschau im Modal |
| Abo-Verträge ändern/löschen/hinzufügen | AboVerwaltung (vollständiges CRUD-Interface) |

### ✅ Soll-Kriterien
| Kriterium | Abgedeckt in |
|---|---|
| Monatsübersicht der Einnahmen | AbrechnungsUebersicht → Tab "Monatsübersicht" |
| Jahresübersicht der Einnahmen | AbrechnungsUebersicht → Tab "Jahresübersicht" + Balkendiagramm |
| Sonderangebote einberechnen | AboVerwaltung (Feld `sonderangebot`), AbrechnungsUebersicht (Spalte) |

### ✅ Kann-Kriterien
| Kriterium | Abgedeckt in |
|---|---|
| Export der Übersicht (CSV/PDF) | AbrechnungsUebersicht (Buttons vorhanden, Logik to-do) |

### ❌ Abgrenzungskriterien (bewusst nicht implementiert)
- Keine automatische Abrechnung beim Kunden
- Keine Bilanz außerhalb des Kundenbezugs
- Kein automatischer E-Mail-Versand
- Keine Benachrichtigungsfunktion bei Abo-Auslauf

---

## Datenmodell-Mapping (lt. Pflichtenheft)

```js
// Abo-Objekt (aus API)
{
  aboNr: 'A001',           // PK
  bezeichnung: 'Premium',
  grundpreis: 49.90,
  kuendigungsfrist: '4 Wochen',
  kurse: true,             // Ja/Nein
  getraenke: false,        // Ja/Nein
  sauna: true,             // Ja/Nein
  sonderangebot: ''        // z.B. "Sommeraktion -10%"
}

// Kunden-Objekt (aus API)
{
  kundenNr: 'K001',        // PK
  vorname: 'Max',
  nachname: 'Mustermann',
  adresse: 'Musterstr. 1',
  iban: 'DE89370400440532013000',  // FK → Kontotabelle
  aboNr: 'A001',           // FK
  ermaessigungssatz: 10    // in %
}

// Abrechnungs-Objekt (aus API)
{
  abrechnungsNr: 'ABR001', // PK
  kundenNr: 'K001',        // FK
  aboNr: 'A001',           // FK
  ermaessigungssatz: 10,
  grundpreis: 49.90,
  endbetrag: 44.91,        // Trigger berechnet automatisch
  monat: '2026-05'
}
```

---

## Testplan (lt. Pflichtenheft Abschnitt 2b.2)

| Nr | Testfall | Zu testen in |
|---|---|---|
| T1 | Kundenabrechnung erstellen | KundenVerwaltung → AbrechnungsUebersicht |
| T2 | Abo-Vertrag bearbeiten | AboVerwaltung → Grundpreis ändern |
| T3 | Monatsübersicht laden & exportieren | AbrechnungsUebersicht → Monat → Export |
