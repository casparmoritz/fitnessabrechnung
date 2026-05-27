# FitnessAbrechnung – Vollständige Anleitung

## Projektstruktur

```
FitnessAbrechnung_Komplett/
├── backend/               ← C# ASP.NET Core 9 Web-API
│   ├── Controllers/       ← REST-Endpunkte (6 Controller)
│   ├── Models/            ← Entity Framework Modelle (7 Tabellen)
│   ├── Data/              ← AppDbContext (DB-Konfiguration)
│   ├── Program.cs         ← Startup, CORS, Oracle-Verbindung
│   ├── appsettings.json   ← Oracle-Verbindungsstring ← HIER ANPASSEN!
│   └── csharp-webapi.csproj
├── frontend/              ← Vue.js 2 (Vue CLI)
│   └── src/
│       ├── App.vue        ← Hauptkomponente, Navigation, API-Verbindung
│       └── components/
│           ├── KundenVerwaltung.vue
│           ├── AboVerwaltung.vue
│           ├── AbrechnungsUebersicht.vue
│           └── EinstellungenView.vue
└── datenbank/
    └── schema_und_testdaten.sql  ← Oracle-Schema + Testdaten
```

---

## 1. Datenbank einrichten

1. Öffne SQL*Plus, DBeaver oder Oracle SQL Developer
2. Verbinde dich mit dem Oracle-Server: `db-server.s-atiw.de:1521/atiwora`
3. Führe `datenbank/schema_und_testdaten.sql` vollständig aus

---

## 2. Backend starten (C# API)

```bash
cd backend/

# Pakete wiederherstellen
dotnet restore

# Starten (Port 5000 HTTP, 5001 HTTPS)
dotnet run
```

> Die API läuft dann auf: `http://localhost:5000`  
> API-Dokumentation (Scalar): `http://localhost:5000/scalar/v1`

**Oracle-Verbindung anpassen** (`appsettings.json`):
```json
"OracleConnection": "User Id=DEIN_USER;Password=DEIN_PASSWORT;Data Source=HOST:PORT/SERVICE;"
```

---

## 3. Frontend starten (Vue.js)

```bash
cd frontend/

# Abhängigkeiten installieren (nur beim ersten Mal)
npm install

# Entwicklungsserver starten
npm run serve
```

> Vue läuft dann auf: `http://localhost:8080`

---

## 4. Verbindung konfigurieren

Im Browser unter **Einstellungen** (Zahnrad-Icon in der Sidebar):
- API-URL eintragen: `http://localhost:5000/api`
- **Speichern & Testen** klicken

---

## Behobene Fehler gegenüber dem Original

| Problem | Ursache | Fix |
|---|---|---|
| `GET /api/ermaessigungen` → 404 | `ErmaessigteController` fehlte komplett | Neuer Controller erstellt |
| Vue zeigt immer Testdaten | Alle 3 API-Anfragen mussten OK sein | Controller ergänzt + CORS-Ports angepasst |
| JSON-Fehler (circular reference) | Navigation-Properties ohne `JsonIgnore` | `ReferenceHandler.IgnoreCycles` in Program.cs |
| `TblAbrechnung` INSERT schlägt fehl | `ABRECHNUNGSMONAT` NOT NULL fehlt im Model | Feld in Model + AppDbContext ergänzt |
| `TblKundenAbo` nicht im Model | Tabelle in SQL aber nicht in C# | `TblKundenAbo.cs` + DbSet erstellt |
| `kurs`/`getraenke` als `bool` statt `int` | Oracle NUMBER(1) ≠ bool | In Model zu `int` geändert, Vue nutzt `:true-value="1"` |
| CORS blockiert Vue CLI (Port 8080) | Nur Ports 5173 und 3000 erlaubt | Port 8080 + 127.0.0.1 Varianten ergänzt |
| `kuendigsfrist` Feldname | C# hatte `Kuendigsfrist`, SQL `KUENDINGSFRIST` | Vereinheitlicht, Columnname explizit gesetzt |
| API-URL hardcoded `10.211.55.5` | Schul-IP, funktioniert nicht lokal | localStorage-basiert, Standard `localhost:5000` |
