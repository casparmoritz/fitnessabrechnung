<template>
  <!-- ==========================================
    APP SHELL – Fitti Exempel GmbH
    Pflichtenheft v1.2 · JCR IT Solution GmbH
    ========================================== -->
  <div id="app-shell">

    <!-- ── SIDEBAR ── -->
    <aside class="sidebar">
      <!-- Logo -->
      <div class="sidebar-logo">
        <div class="logo-icon">💪</div>
        <h1>FittiAbrechnung</h1>
        <span>Fitti Exempel GmbH</span>
      </div>

      <!-- Navigation -->
      <nav class="sidebar-nav">
        <div class="nav-section-label">Verwaltung</div>

        <!-- Kundenverwaltung (Muss-Kriterium) -->
        <div
          class="nav-item"
          :class="{ active: activeView === 'kunden' }"
          @click="navigate('kunden')"
          id="nav-kunden"
        >
          <span class="nav-icon">👤</span>
          Kundenverwaltung
        </div>

        <!-- Abo-Verwaltung (Muss-Kriterium) -->
        <div
          class="nav-item"
          :class="{ active: activeView === 'abos' }"
          @click="navigate('abos')"
          id="nav-abos"
        >
          <span class="nav-icon">📋</span>
          Abo-Verwaltung
        </div>

        <div class="nav-section-label">Finanzen</div>

        <!-- Abrechnungsübersicht (Soll-Kriterium) -->
        <div
          class="nav-item"
          :class="{ active: activeView === 'abrechnungen' }"
          @click="navigate('abrechnungen')"
          id="nav-abrechnungen"
        >
          <span class="nav-icon">📊</span>
          Abrechnungen
        </div>


        <div class="nav-section-label">System</div>

        <!-- Einstellungen -->
        <div
          class="nav-item"
          :class="{ active: activeView === 'einstellungen' }"
          @click="navigate('einstellungen')"
          id="nav-einstellungen"
        >
          <span class="nav-icon">⚙️</span>
          Einstellungen
        </div>
      </nav>

      <!-- Footer -->
      <div class="sidebar-footer">
        <div>v1.2 · JCR IT Solution</div>
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

        <!-- API-Verbindungsstatus-Anzeige -->
        <div class="api-status-indicator" :class="apiStatus" @click="ladeStammdaten" title="Verbindung neu laden">
          <span class="status-dot"></span>
          <span class="status-text">{{ apiStatusText }}</span>
          <span class="status-refresh">🔄</span>
        </div>
      </header>

      <!-- Status-Banner bei Verbindungsschwierigkeiten -->
      <div v-if="apiStatus === 'offline'" class="api-banner offline">
        <span>⚠️ <strong>API Offline:</strong> Keine Verbindung zum Server ({{ apiBaseUrl }}). Es werden lokale Testdaten verwendet. Änderungen werden nur im Browser gespeichert.</span>
        <button class="btn btn-sm btn-ghost" @click="ladeStammdaten" style="margin-left: auto;">Erneut versuchen</button>
      </div>
      <div v-if="apiStatus === 'online' && showSuccessMessage" class="api-banner online">
        <span>✅ <strong>API Verbunden:</strong> Erfolgreich mit dem Server ({{ apiBaseUrl }}) verbunden und Stammdaten geladen.</span>
        <button class="btn-close-banner" @click="showSuccessMessage = false" style="margin-left: auto; background: transparent; border: none; color: inherit; cursor: pointer; font-size: 16px;">✕</button>
      </div>

      <!-- Toast Benachrichtigungen -->
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

      <!-- PAGE VIEWS -->
      <main class="page-container">

        <!-- Kundenverwaltung -->
        <KundenVerwaltung
          v-if="activeView === 'kunden'"
          :kunden="kunden"
          :abos="abos"
          :ermaessigungen="ermaessigungen"
          @kunden-updated="onKundenUpdated"
          @show-toast="triggerToast"
        />

        <!-- Abo-Verwaltung -->
        <AboVerwaltung
          v-if="activeView === 'abos'"
          :abos="abos"
          @abos-updated="onAbosUpdated"
          @show-toast="triggerToast"
        />

        <!-- Abrechnungsübersicht -->
        <AbrechnungsUebersicht
          v-if="activeView === 'abrechnungen'"
          :kunden="kunden"
          :abos="abos"
          :ermaessigungen="ermaessigungen"
        />


        <!-- Einstellungen -->
        <EinstellungenView
          v-if="activeView === 'einstellungen'"
        />

      </main>
    </div>
  </div>
</template>

<script>
/* =========================================================
   JAVASCRIPT – App.vue
   Haupt-Komponente: Sidebar-Navigation & Datenhaltung
   ========================================================= */

// Die vier Unterseiten werden als eigene Komponenten eingebunden
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

  // data() liefert alle reaktiven Variablen der Komponente
  data() {
    return {
      // Welche Seite gerade angezeigt wird (Schlüssel: 'kunden', 'abos', 'abrechnungen', 'einstellungen')
      activeView: 'kunden',

      // Überschriften für die Topbar je nach aktiver Ansicht
      viewTitles: {
        kunden:       'Kundenverwaltung',
        abos:         'Abo-Verwaltung',
        abrechnungen: 'Abrechnungsübersicht',
        einstellungen:'Einstellungen'
      },

      // Untertitel für die Topbar je nach aktiver Ansicht
      viewSubtitles: {
        kunden:       'Kunden anlegen, bearbeiten & Abos zuweisen',
        abos:         'Abotypen verwalten, Preise & Optionen',
        abrechnungen: 'Monats- & Jahresübersichten der Einnahmen',
        einstellungen:'Systeminformationen & API-Konfiguration'
      },

      // Reaktiv gehaltene Stammdaten – starten leer und werden per API oder Testdaten geladen
      kunden: [],
      abos: [],
      ermaessigungen: [],

      // API Verbindungsstatus ('loading', 'online', 'offline')
      apiStatus: 'loading',
      showSuccessMessage: false,
      apiBaseUrl: '',

      // Array für aktive Toast-Meldungen im UI
      toasts: [],

      // Fallback-Testdaten bei fehlgeschlagener API-Verbindung
      testKunden: [
        {
          kundenNr: 'K001',
          vorname: 'Max',
          nachname: 'Mustermann',
          adresse: 'Musterstr. 1, 12345 Musterstadt',
          iban: 'DE89370400440532013000',
          aboNr: 'A001',
          ermaessigungssatz: 10
        },
        {
          kundenNr: 'K002',
          vorname: 'Erika',
          nachname: 'Mustermann',
          adresse: 'Musterweg 2, 12345 Musterstadt',
          iban: 'DE89370400440532013001',
          aboNr: 'A002',
          ermaessigungssatz: 0
        }
      ],
      testAbos: [
        {
          aboNr: 'A001',
          bezeichnung: 'Premium-Abo',
          grundpreis: 49.90,
          kuendigungsfrist: '2026-12-31',
          kurs: true,
          getraenke: true
        },
        {
          aboNr: 'A002',
          bezeichnung: 'Standard-Abo',
          grundpreis: 29.90,
          kuendigungsfrist: '2026-06-30',
          kurs: false,
          getraenke: false
        }
      ],
      testErmaessigungen: [
        { ermID: 'ERM01', bezeichnung: 'Student/Azubi', ermaessigungssatz: 10 },
        { ermID: 'ERM02', bezeichnung: 'Senior', ermaessigungssatz: 15 }
      ]
    }
  },

  // computed: werden automatisch neu berechnet wenn abhängige Daten sich ändern
  computed: {
    // Gibt das aktuelle Datum als deutschen String zurück (z.B. "Do., 21. Mai 2026")
    currentDate() {
      return new Date().toLocaleDateString('de-DE', {
        weekday: 'short', day: '2-digit', month: 'short', year: 'numeric'
      })
    },

    // Gibt die Übersetzung des API-Verbindungsstatus für die Anzeige zurück
    apiStatusText() {
      if (this.apiStatus === 'loading') return 'Verbinde...'
      if (this.apiStatus === 'online') return 'API Online'
      return 'Offline (Testdaten)'
    }
  },

  // mounted() wird einmalig aufgerufen sobald die App im Browser angezeigt wird
  mounted() {
    this.ladeStammdaten()
  },

  methods: {
    // Wechselt die angezeigte Seite in der Sidebar
    navigate(view) {
      this.activeView = view
    },

    // Wird von KundenVerwaltung aufgerufen wenn Kundendaten geändert wurden
    onKundenUpdated(updatedKunden) {
      this.kunden = updatedKunden
    },

    // Wird von AboVerwaltung aufgerufen wenn Abodaten geändert wurden
    onAbosUpdated(updatedAbos) {
      this.abos = updatedAbos
    },

    // Fügt eine neue Toast-Meldung hinzu und entfernt diese nach 4 Sekunden automatisch
    triggerToast(toast) {
      const id = Date.now()
      this.toasts.push({
        id,
        text: toast.text || '',
        type: toast.type || 'info'
      })
      setTimeout(() => {
        this.toasts = this.toasts.filter(t => t.id !== id)
      }, 4000)
    },

    // Lädt Kunden, Abos und Ermäßigungen parallel von der API
    async ladeStammdaten() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      this.apiBaseUrl = baseUrl
      this.apiStatus = 'loading'
      this.showSuccessMessage = false

      try {
        const [resKunden, resAbos, resErmaessigungen] = await Promise.all([
          fetch(`${baseUrl}/kunden`),
          fetch(`${baseUrl}/abos`),
          fetch(`${baseUrl}/ermaessigungen`)
        ])

        if (resKunden.ok && resAbos.ok && resErmaessigungen.ok) {
          this.kunden = await resKunden.json()
          this.abos = await resAbos.json()
          this.ermaessigungen = await resErmaessigungen.json()
          
          this.apiStatus = 'online'
          this.showSuccessMessage = true
          this.triggerToast({
            text: 'Verbindung zur API hergestellt. Daten geladen.',
            type: 'success'
          })
          
          // Erfolgsmeldung nach 5 Sekunden automatisch ausblenden
          setTimeout(() => {
            this.showSuccessMessage = false
          }, 5000)
        } else {
          throw new Error(`Verbindung erfolgreich aber Antwort fehlerhaft.`)
        }
      } catch (err) {
        console.warn('API ist offline, nutze lokale Testdaten als Fallback:', err)
        this.kunden = [...this.testKunden]
        this.abos = [...this.testAbos]
        this.ermaessigungen = [...this.testErmaessigungen]
        this.apiStatus = 'offline'
        this.triggerToast({
          text: 'API ist offline. Testdaten geladen.',
          type: 'warning'
        })
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

/* ── API Verbindungsstatus in der Topbar ── */
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
.api-status-indicator:hover {
  border-color: var(--clr-primary);
}
.api-status-indicator .status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
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
  font-size: 10px;
  opacity: 0.6;
  margin-left: 2px;
  transition: transform var(--tr-fast);
}
.api-status-indicator:hover .status-refresh {
  transform: rotate(180deg);
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.3); opacity: 0.5; }
}
@keyframes pulse-warn {
  0%, 100% { transform: scale(1); opacity: 1; box-shadow: 0 0 8px var(--clr-warning); }
  50% { transform: scale(1.3); opacity: 0.4; box-shadow: 0 0 2px var(--clr-warning); }
}

/* ── API Status Banners ── */
.api-banner {
  display: flex;
  align-items: center;
  gap: var(--sp-md);
  padding: var(--sp-sm) var(--sp-lg);
  font-size: 13px;
  font-weight: 500;
  border-bottom: 1px solid;
  animation: slideDown 250ms ease;
}
.api-banner.offline {
  background: var(--clr-warning-dim);
  border-color: rgba(255,182,39,0.25);
  color: var(--clr-warning);
}
.api-banner.online {
  background: var(--clr-success-dim);
  border-color: rgba(54,211,153,0.25);
  color: var(--clr-success);
}
@keyframes slideDown {
  from { transform: translateY(-100%); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

/* ── Toast Container & Meldungen ── */
.toast-container {
  position: fixed;
  bottom: var(--sp-lg);
  right: var(--sp-lg);
  z-index: 999;
  display: flex;
  flex-direction: column;
  gap: var(--sp-sm);
  max-width: 380px;
}
.toast-msg {
  display: flex;
  align-items: center;
  gap: var(--sp-sm);
  padding: 12px 18px;
  background: var(--clr-surface);
  border: 1px solid var(--clr-border);
  border-radius: var(--rad-md);
  box-shadow: var(--shadow-lg);
  color: var(--clr-text);
  font-size: 13px;
  font-weight: 500;
  animation: toastIn 300ms cubic-bezier(0.68, -0.55, 0.27, 1.55);
}
.toast-msg.success { border-left: 4px solid var(--clr-success); }
.toast-msg.warning { border-left: 4px solid var(--clr-warning); }
.toast-msg.danger  { border-left: 4px solid var(--clr-danger); }
.toast-msg.info    { border-left: 4px solid var(--clr-primary); }

@keyframes toastIn {
  from { transform: translateX(100%) scale(0.9); opacity: 0; }
  to { transform: translateX(0) scale(1); opacity: 1; }
}
</style>
