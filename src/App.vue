<template>
  <!-- ==========================================
    APP SHELL – Fitti Exempel GmbH
    Hauptkomponente: Sidebar-Navigation & Datenhaltung
    ========================================== -->
  <div id="app-shell">

    <!-- ── SIDEBAR ── -->
    <aside class="sidebar">
      <div class="sidebar-logo">
        <div class="logo-icon">💪</div>
        <h1>FittiAbrechnung</h1>
        <span>Fitti Exempel GmbH</span>
      </div>

      <nav class="sidebar-nav">
        <div class="nav-section-label">Verwaltung</div>

        <div class="nav-item" :class="{ active: activeView === 'kunden' }" @click="navigate('kunden')" id="nav-kunden">
          <span class="nav-icon">👤</span> Kundenverwaltung
        </div>
        <div class="nav-item" :class="{ active: activeView === 'abos' }" @click="navigate('abos')" id="nav-abos">
          <span class="nav-icon">📋</span> Abo-Verwaltung
        </div>

        <div class="nav-section-label">Finanzen</div>

        <div class="nav-item" :class="{ active: activeView === 'abrechnungen' }" @click="navigate('abrechnungen')" id="nav-abrechnungen">
          <span class="nav-icon">📊</span> Abrechnungen
        </div>

        <div class="nav-section-label">System</div>

        <div class="nav-item" :class="{ active: activeView === 'einstellungen' }" @click="navigate('einstellungen')" id="nav-einstellungen">
          <span class="nav-icon">⚙️</span> Einstellungen
        </div>
      </nav>

      <div class="sidebar-footer">
        <div>v1.3 · JCR IT Solution</div>
        <!-- JavaScript: Datum wird reaktiv berechnet (computed: currentDate) -->
        <div style="margin-top:4px;">{{ currentDate }}</div>
      </div>
    </aside>

    <!-- ── MAIN CONTENT ── -->
    <div class="main-content">

      <!-- Top Bar -->
      <header class="topbar">
        <span class="topbar-title">
          {{ viewTitles[activeView] }}
          <span class="topbar-subtitle">{{ viewSubtitles[activeView] }}</span>
        </span>

        <!-- JavaScript: API-Status-Anzeige mit reaktivem CSS-Klassen-Binding -->
        <div class="api-status-indicator" :class="apiStatus" @click="ladeStammdaten" title="Verbindung neu laden">
          <span class="status-dot"></span>
          <span class="status-text">{{ apiStatusText }}</span>
          <span class="status-refresh">🔄</span>
        </div>
      </header>

      <!-- Offline-Banner -->
      <div v-if="apiStatus === 'offline'" class="api-banner offline">
        <span>⚠️ <strong>API Offline:</strong> Keine Verbindung zum Server ({{ apiBaseUrl }}). Es werden lokale Testdaten verwendet.</span>
        <!-- JavaScript: @click ruft ladeStammdaten() auf -->
        <button class="btn btn-sm btn-ghost" @click="ladeStammdaten" style="margin-left: auto;">Erneut versuchen</button>
      </div>
      <div v-if="apiStatus === 'online' && showSuccessMessage" class="api-banner online">
        <span>✅ <strong>API Verbunden:</strong> Erfolgreich mit dem Server verbunden.</span>
        <!-- JavaScript: @click setzt showSuccessMessage auf false -->
        <button class="btn-close-banner" @click="showSuccessMessage = false" style="margin-left:auto;background:transparent;border:none;color:inherit;cursor:pointer;font-size:16px;">✕</button>
      </div>

      <!-- Toast Benachrichtigungen -->
      <!-- JavaScript: v-for rendert jede aktive Toast-Meldung aus dem toasts-Array -->
      <div class="toast-container">
        <div v-for="toast in toasts" :key="toast.id" class="toast-msg" :class="toast.type">
          <span class="toast-icon">
            <span v-if="toast.type === 'success'">✅</span>
            <span v-else-if="toast.type === 'warning'">⚠️</span>
            <span v-else-if="toast.type === 'danger'">❌</span>
            <span v-else>ℹ️</span>
          </span>
          <span class="toast-text">{{ toast.text }}</span>
        </div>
      </div>

      <!-- PAGE VIEWS: v-if schaltet die aktive Seite ein -->
      <main class="page-container">

        <!-- JavaScript: props werden per :prop-name="variable" übergeben -->
        <KundenVerwaltung
          v-if="activeView === 'kunden'"
          :kunden="kunden"
          :abos="abos"
          :ermaessigungen="ermaessigungen"
          :konten="konten"
          :banken="banken"
          @kunden-updated="onKundenUpdated"
          @show-toast="triggerToast"
        />

        <AboVerwaltung
          v-if="activeView === 'abos'"
          :abos="abos"
          @abos-updated="onAbosUpdated"
          @show-toast="triggerToast"
        />

        <AbrechnungsUebersicht
          v-if="activeView === 'abrechnungen'"
          :kunden="kunden"
          :abos="abos"
          :ermaessigungen="ermaessigungen"
        />

        <EinstellungenView
          v-if="activeView === 'einstellungen'"
        />

      </main>
    </div>
  </div>
</template>

<script>
/* =========================================================
   ██ JAVASCRIPT – App.vue
   Haupt-Komponente: Sidebar-Navigation, API-Verbindung & Datenhaltung
   Alle API-Felder verwenden camelCase (Standard von .NET JSON)
   ========================================================= */

import KundenVerwaltung      from './components/KundenVerwaltung.vue'
import AboVerwaltung         from './components/AboVerwaltung.vue'
import AbrechnungsUebersicht from './components/AbrechnungsUebersicht.vue'
import EinstellungenView     from './components/EinstellungenView.vue'

export default {
  name: 'App',

  // Alle importierten Komponenten müssen hier registriert werden
  components: {
    KundenVerwaltung,
    AboVerwaltung,
    AbrechnungsUebersicht,
    EinstellungenView
  },

  /* ── data() ─────────────────────────────────────────────────────────────
     Alle reaktiven Variablen der Komponente.
     WICHTIG: Die API liefert camelCase (z.B. "kundennr", "abonr", "ermid")
     weil ASP.NET Core JSON-Serialisierung standardmäßig camelCase verwendet.
  ─────────────────────────────────────────────────────────────────────── */
  data() {
    return {
      // Aktuell angezeigte Seite
      activeView: 'kunden',

      // Überschriften für die Topbar
      viewTitles: {
        kunden:        'Kundenverwaltung',
        abos:          'Abo-Verwaltung',
        abrechnungen:  'Abrechnungsübersicht',
        einstellungen: 'Einstellungen'
      },

      viewSubtitles: {
        kunden:        'Kunden anlegen, bearbeiten & Abos zuweisen',
        abos:          'Abotypen verwalten, Preise & Optionen',
        abrechnungen:  'Monats- & Jahresübersichten der Einnahmen',
        einstellungen: 'Systeminformationen & API-Konfiguration'
      },

      // ── Stammdaten (werden per API geladen) ──────────────────────────
      // Feldnamen entsprechen der C# camelCase-Serialisierung:
      // TblKunde  → { kundennr, vorname, nachname, iban, konto: { bic, bank: { bankName } } }
      // TblAbo    → { abonr, grundpreis, kuendigsfrist, kurs, getraenke, laufzeit }
      // TblErm.   → { ermid, ermaessigungssatz }
      // TblKonto  → { iban, bic, bank: { bic, bankName } }
      // TblBank   → { bic, bankName }
      kunden:         [],
      abos:           [],
      ermaessigungen: [],
      konten:         [],
      banken:         [],

      // API Verbindungsstatus: 'loading' | 'online' | 'offline'
      apiStatus:          'loading',
      showSuccessMessage: false,
      apiBaseUrl:         '',

      // Toast-Meldungen (werden nach 4 Sekunden automatisch entfernt)
      toasts: [],

      // ── Fallback-Testdaten (wenn API nicht erreichbar) ───────────────
      testKunden: [
        { kundennr: 1, vorname: 'Max',   nachname: 'Mustermann', iban: 'DE89370400440532013000',
          konto: { bic: 'SPKADEFFFXX', bank: { bankName: 'Sparkasse' } } },
        { kundennr: 2, vorname: 'Erika', nachname: 'Musterfrau',  iban: 'DE22100700000123456789',
          konto: { bic: 'DEUTDEDDFXX', bank: { bankName: 'Deutsche Bank' } } }
      ],
      testAbos: [
        { abonr: 1, grundpreis: 49.99, kuendigsfrist: '2026-12-31T00:00:00Z', kurs: 1, getraenke: 1, laufzeit: '1-0' },
        { abonr: 2, grundpreis: 19.99, kuendigsfrist: '2026-06-30T00:00:00Z', kurs: 0, getraenke: 0, laufzeit: '0-6' }
      ],
      testErmaessigungen: [
        { ermid: 1, ermaessigungssatz: 0.20 },
        { ermid: 2, ermaessigungssatz: 0.00 }
      ],
      testKonten: [
        { iban: 'DE89370400440532013000', bic: 'SPKADEFFFXX', bank: { bankName: 'Sparkasse' } },
        { iban: 'DE22100700000123456789', bic: 'DEUTDEDDFXX', bank: { bankName: 'Deutsche Bank' } }
      ],
      testBanken: [
        { bic: 'SPKADEFFFXX', bankName: 'Sparkasse' },
        { bic: 'DEUTDEDDFXX', bankName: 'Deutsche Bank' }
      ]
    }
  },

  // ── computed ─────────────────────────────────────────────────────────────
  computed: {
    // Aktuelles Datum als deutschsprachiger String
    currentDate() {
      return new Date().toLocaleDateString('de-DE', {
        weekday: 'short', day: '2-digit', month: 'short', year: 'numeric'
      })
    },

    // Anzeigetext für den API-Verbindungsstatus
    apiStatusText() {
      if (this.apiStatus === 'loading') return 'Verbinde...'
      if (this.apiStatus === 'online')  return 'API Online'
      return 'Offline (Testdaten)'
    }
  },

  // mounted() wird einmalig aufgerufen sobald die Komponente im DOM ist
  mounted() {
    this.ladeStammdaten()
  },

  methods: {
    // Wechselt die angezeigte Seite in der Sidebar
    navigate(view) {
      this.activeView = view
    },

    // Event-Handler: Wird aufgerufen wenn KundenVerwaltung Daten geändert hat
    onKundenUpdated(updatedKunden) {
      this.kunden = updatedKunden
    },

    // Event-Handler: Wird aufgerufen wenn AboVerwaltung Daten geändert hat
    onAbosUpdated(updatedAbos) {
      this.abos = updatedAbos
    },

    // Toast-Meldung anzeigen (verschwindet nach 4 Sekunden)
    triggerToast(toast) {
      const id = Date.now()
      this.toasts.push({ id, text: toast.text || '', type: toast.type || 'info' })
      setTimeout(() => {
        this.toasts = this.toasts.filter(t => t.id !== id)
      }, 4000)
    },

    /* ── ladeStammdaten() ─────────────────────────────────────────────
       Lädt alle Stammdaten parallel von der C# API.
       API-URL wird aus dem localStorage gelesen (konfigurierbar in Einstellungen).
       Fallback: Testdaten wenn die API nicht erreichbar ist.
    ─────────────────────────────────────────────────────────────────── */
    async ladeStammdaten() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      this.apiBaseUrl = baseUrl
      this.apiStatus  = 'loading'
      this.showSuccessMessage = false

      try {
        // Alle 5 Endpunkte parallel abrufen (Promise.all)
        const [resKunden, resAbos, resErm, resKonten, resBanken] = await Promise.all([
          fetch(`${baseUrl}/kunden`),
          fetch(`${baseUrl}/abos`),
          fetch(`${baseUrl}/ermaessigungen`),   // ← Neuer Controller!
          fetch(`${baseUrl}/konten`),
          fetch(`${baseUrl}/banken`)
        ])

        // Alle Antworten müssen OK sein (Status 200-299)
        if (resKunden.ok && resAbos.ok && resErm.ok && resKonten.ok && resBanken.ok) {
          this.kunden         = await resKunden.json()
          this.abos           = await resAbos.json()
          this.ermaessigungen = await resErm.json()
          this.konten         = await resKonten.json()
          this.banken         = await resBanken.json()

          this.apiStatus = 'online'
          this.showSuccessMessage = true
          this.triggerToast({ text: 'Verbindung zur API hergestellt. Stammdaten geladen.', type: 'success' })

          // Erfolgsmeldung nach 5 Sekunden ausblenden
          setTimeout(() => { this.showSuccessMessage = false }, 5000)
        } else {
          throw new Error('Mindestens ein API-Endpunkt antwortete mit einem Fehler.')
        }
      } catch (err) {
        console.warn('API ist offline, nutze lokale Testdaten:', err)
        this.kunden         = [...this.testKunden]
        this.abos           = [...this.testAbos]
        this.ermaessigungen = [...this.testErmaessigungen]
        this.konten         = [...this.testKonten]
        this.banken         = [...this.testBanken]
        this.apiStatus = 'offline'
        this.triggerToast({ text: 'API ist offline. Testdaten geladen.', type: 'warning' })
      }
    }
  }
}
</script>

<style>
@import './assets/main.css';

#app {
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

#app-shell {
  display: flex;
  flex: 1;
  width: 100%;
  min-height: 100vh;
  background: var(--clr-bg);
}

.page-container {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

/* ── API Verbindungsstatus ── */
.api-status-indicator {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--clr-surface-2);
  border: 1px solid var(--clr-border);
  padding: 6px 12px;
  border-radius: var(--rad-pill);
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  user-select: none;
  transition: all var(--tr-fast);
}
.api-status-indicator:hover { border-color: var(--clr-primary); }
.api-status-indicator .status-dot {
  width: 8px; height: 8px; border-radius: 50%;
}
.api-status-indicator.loading .status-dot {
  background: var(--clr-info);
  box-shadow: 0 0 8px var(--clr-info);
  animation: pulse 1.5s infinite ease-in-out;
}
.api-status-indicator.online .status-dot {
  background: var(--clr-success);
  box-shadow: 0 0 8px var(--clr-success);
}
.api-status-indicator.offline .status-dot {
  background: var(--clr-warning);
  box-shadow: 0 0 8px var(--clr-warning);
  animation: pulse-warn 1.5s infinite ease-in-out;
}
.api-status-indicator .status-refresh {
  font-size: 10px; opacity: 0.6; margin-left: 2px;
  transition: transform var(--tr-fast);
}
.api-status-indicator:hover .status-refresh { transform: rotate(180deg); }

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.3); opacity: 0.5; }
}
@keyframes pulse-warn {
  0%, 100% { transform: scale(1); opacity: 1; box-shadow: 0 0 8px var(--clr-warning); }
  50% { transform: scale(1.3); opacity: 0.4; box-shadow: 0 0 2px var(--clr-warning); }
}

/* ── API Banners ── */
.api-banner {
  display: flex; align-items: center; gap: var(--sp-md);
  padding: var(--sp-sm) var(--sp-lg);
  font-size: 13px; font-weight: 500;
  border-bottom: 1px solid;
  animation: slideDown 250ms ease;
}
.api-banner.offline { background: var(--clr-warning-dim); border-color: rgba(255,182,39,0.25); color: var(--clr-warning); }
.api-banner.online  { background: var(--clr-success-dim); border-color: rgba(54,211,153,0.25); color: var(--clr-success); }
@keyframes slideDown {
  from { transform: translateY(-100%); opacity: 0; }
  to   { transform: translateY(0);     opacity: 1; }
}

/* ── Toasts ── */
.toast-container {
  position: fixed; bottom: var(--sp-lg); right: var(--sp-lg);
  z-index: 999; display: flex; flex-direction: column;
  gap: var(--sp-sm); max-width: 380px;
}
.toast-msg {
  display: flex; align-items: center; gap: var(--sp-sm);
  padding: 12px 18px;
  background: var(--clr-surface); border: 1px solid var(--clr-border);
  border-radius: var(--rad-md); box-shadow: var(--shadow-lg);
  color: var(--clr-text); font-size: 13px; font-weight: 500;
  animation: toastIn 300ms cubic-bezier(0.68,-0.55,0.27,1.55);
}
.toast-msg.success { border-left: 4px solid var(--clr-success); }
.toast-msg.warning { border-left: 4px solid var(--clr-warning); }
.toast-msg.danger  { border-left: 4px solid var(--clr-danger); }
.toast-msg.info    { border-left: 4px solid var(--clr-primary); }
@keyframes toastIn {
  from { transform: translateX(100%) scale(0.9); opacity: 0; }
  to   { transform: translateX(0)     scale(1);  opacity: 1; }
}
</style>
