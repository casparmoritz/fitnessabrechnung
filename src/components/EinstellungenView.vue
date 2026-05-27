<template>
  <!-- ==========================================
    EinstellungenView.vue
    Systemeinstellungen: API-URL anpassen, Verbindungstest
    ========================================== -->
  <div class="page">
    <div class="page-header">
      <div>
        <h2 class="page-title">Einstellungen</h2>
        <p class="page-desc">API-Verbindung konfigurieren und Systeminformationen anzeigen</p>
      </div>
    </div>

    <div class="card">
      <div class="card-header">
        <div class="card-title">🔌 API-Verbindung</div>
        <div class="card-subtitle">URL der C# ASP.NET Core Web-API</div>
      </div>

      <div class="form-grid" style="padding:var(--sp-lg);">
        <div class="form-group full">
          <label for="input-api-url">API Basis-URL</label>
          <!-- JavaScript: v-model bindet apiUrl-Feld bidirektional -->
          <input id="input-api-url" type="text" v-model="apiUrl"
                 placeholder="http://localhost:5000/api" />
          <small style="color:var(--clr-text-muted);margin-top:4px;display:block;">
            Standard: <code>http://localhost:5000/api</code> (lokale Entwicklung) |
            Schule: <code>http://10.211.55.5:5000/api</code>
          </small>
        </div>
      </div>

      <div style="padding:0 var(--sp-lg) var(--sp-lg);display:flex;gap:var(--sp-sm);">
        <!-- JavaScript: @click speichert URL in localStorage und löst Verbindungstest aus -->
        <button class="btn btn-primary" @click="speichereUrl">💾 Speichern & Testen</button>
        <button class="btn btn-ghost"   @click="resetUrl">↩ Zurücksetzen</button>
      </div>

      <!-- JavaScript: v-if zeigt Testergebnis an -->
      <div v-if="testErgebnis" class="alert" :class="'alert-' + testTyp" style="margin:0 var(--sp-lg) var(--sp-lg);">
        {{ testErgebnis }}
      </div>
    </div>

    <!-- Systeminformationen -->
    <div class="card mt-lg">
      <div class="card-header">
        <div class="card-title">ℹ️ Systeminformationen</div>
      </div>
      <div style="padding:var(--sp-lg);">
        <table style="width:100%;font-size:13px;">
          <tbody>
            <tr><td style="padding:6px 0;color:var(--clr-text-muted);width:200px;">Frontend</td><td>Vue.js 2.x (Vue CLI)</td></tr>
            <tr><td style="padding:6px 0;color:var(--clr-text-muted);">Backend</td><td>ASP.NET Core 9 Web-API (C#)</td></tr>
            <tr><td style="padding:6px 0;color:var(--clr-text-muted);">ORM</td><td>Entity Framework Core + Oracle.EntityFrameworkCore</td></tr>
            <tr><td style="padding:6px 0;color:var(--clr-text-muted);">Datenbank</td><td>Oracle DB (ATIWORA)</td></tr>
            <tr><td style="padding:6px 0;color:var(--clr-text-muted);">API-Dokumentation</td>
              <td>
                <!-- JavaScript: Computed apiDocsUrl baut URL dynamisch -->
                <a :href="apiDocsUrl" target="_blank" style="color:var(--clr-primary);">
                  {{ apiDocsUrl }} (Scalar API Reference)
                </a>
              </td>
            </tr>
            <tr><td style="padding:6px 0;color:var(--clr-text-muted);">Version</td><td>v1.3 – JCR IT Solution GmbH</td></tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Datenbankschema Übersicht -->
    <div class="card mt-lg">
      <div class="card-header">
        <div class="card-title">🗄️ Datenbankschema</div>
        <div class="card-subtitle">Oracle-Tabellen und ihre Beziehungen</div>
      </div>
      <div style="padding:var(--sp-lg);">
        <div style="display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:var(--sp-md);">
          <div v-for="tabelle in tabellen" :key="tabelle.name" style="border:1px solid var(--clr-border);border-radius:var(--rad-md);padding:var(--sp-md);">
            <div style="font-weight:700;color:var(--clr-primary);margin-bottom:var(--sp-sm);">{{ tabelle.name }}</div>
            <!-- JavaScript: v-for über verschachteltes Array -->
            <div v-for="feld in tabelle.felder" :key="feld" style="font-size:12px;color:var(--clr-text-muted);padding:2px 0;">
              {{ feld }}
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
/* =========================================================
   ██ JAVASCRIPT – EinstellungenView.vue
   API-URL Konfiguration (wird in localStorage gespeichert)
   und Anzeige der Systemarchitektur.
   ========================================================= */

export default {
  name: 'EinstellungenView',

  data() {
    return {
      // API-URL wird aus localStorage gelesen (oder Standard verwendet)
      apiUrl:       localStorage.getItem('apiUrl') || 'http://localhost:5000/api',
      testErgebnis: '',
      testTyp:      'info', // 'success' | 'danger' | 'info'

      // Datenbankschema zur Anzeige
      tabellen: [
        { name: 'TBL_BANK',      felder: ['BIC (PK)', 'BANK'] },
        { name: 'TBL_KONTO',     felder: ['IBAN (PK)', 'BIC (FK→Bank)'] },
        { name: 'TBL_KUNDEN',    felder: ['KUNDENNR (PK)', 'VORNAME', 'NACHNAME', 'IBAN (FK)'] },
        { name: 'TBL_ABO',       felder: ['ABONR (PK)', 'KUENDINGSFRIST', 'KURS', 'GETRAENKE', 'GRUNDPREIS', 'LAUFZEIT'] },
        { name: 'TBL_ERMAESSIGTE', felder: ['ERMID (PK)', 'ERMAESSIGUNGSSATZ'] },
        { name: 'TBL_KUNDENABO', felder: ['KUNDENABONR (PK)', 'KUNDENNR (FK)', 'ABONR (FK)', 'STARTDATUM', 'ENDDATUM', 'STATUS'] },
        { name: 'TBL_ABRECHNUNG', felds: [], felder: ['ABRECHNUNGSNR (PK)', 'KUNDENNR (FK)', 'ABONR (FK)', 'ERMID (FK)', 'KUNDENABONR (FK)', 'RECHNUNGSBETRAG', 'ABRECHNUNGSMONAT'] }
      ]
    }
  },

  computed: {
    // Baut die URL zur API-Dokumentation (Scalar) aus der API-Basis-URL
    apiDocsUrl() {
      const base = this.apiUrl.replace('/api', '')
      return `${base}/scalar/v1`
    }
  },

  methods: {
    // Speichert die API-URL in localStorage und testet die Verbindung
    async speichereUrl() {
      // URL säubern (führende/nachgestellte Leerzeichen entfernen)
      this.apiUrl = this.apiUrl.trim()
      localStorage.setItem('apiUrl', this.apiUrl)
      this.testErgebnis = '⏳ Teste Verbindung...'
      this.testTyp = 'info'

      try {
        // Einfacher Ping auf den /banken-Endpunkt
        const res = await fetch(`${this.apiUrl}/banken`, {
          signal: AbortSignal.timeout(5000) // Timeout nach 5 Sekunden
        })
        if (res.ok) {
          this.testErgebnis = `✅ Verbindung erfolgreich! (HTTP ${res.status})`
          this.testTyp = 'success'
        } else {
          this.testErgebnis = `⚠️ Server antwortet, aber mit Fehlercode ${res.status}.`
          this.testTyp = 'warning'
        }
      } catch (err) {
        this.testErgebnis = `❌ Keine Verbindung: ${err.message}`
        this.testTyp = 'danger'
      }
    },

    // Setzt die API-URL auf den Standardwert zurück
    resetUrl() {
      this.apiUrl = 'http://localhost:5000/api'
      localStorage.setItem('apiUrl', this.apiUrl)
      this.testErgebnis = ''
    }
  }
}
</script>
