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
      </header>

      <!-- PAGE VIEWS -->
      <main class="page-container">

        <!-- Kundenverwaltung -->
        <KundenVerwaltung
          v-if="activeView === 'kunden'"
          :kunden="kunden"
          :abos="abos"
          :ermaessigungen="ermaessigungen"
          @kunden-updated="onKundenUpdated"
        />

        <!-- Abo-Verwaltung -->
        <AboVerwaltung
          v-if="activeView === 'abos'"
          :abos="abos"
          @abos-updated="onAbosUpdated"
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

      /* ✅ IMPLEMENTIERT: Testdaten – werden beim App-Start durch API-Daten ersetzt */
      kunden: [
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
      abos: [
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
      ermaessigungen: [
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
    }
  },

  // ✅ IMPLEMENTIERT: mounted() wird einmalig aufgerufen sobald die App im Browser angezeigt wird
  mounted() {
    this.ladeStammdaten()
  },

  methods: {
    // Wechselt die angezeigte Seite in der Sidebar
    navigate(view) {
      this.activeView = view
    },

    // Wird von KundenVerwaltung aufgerufen wenn Kundendaten geändert wurden
    // Das neue Array wird hier in App.vue gespeichert und an alle Komponenten weitergegeben
    onKundenUpdated(updatedKunden) {
      this.kunden = updatedKunden
    },

    // Wird von AboVerwaltung aufgerufen wenn Abodaten geändert wurden
    onAbosUpdated(updatedAbos) {
      this.abos = updatedAbos
    },

    // ✅ IMPLEMENTIERT: Lädt Kunden, Abos und Ermäßigungen gleichzeitig (parallel) von der API
    async ladeStammdaten() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        // Promise.all: alle drei Requests gleichzeitig starten → spart Zeit
        const [resKunden, resAbos, resErmaessigungen] = await Promise.all([
          fetch(`${baseUrl}/kunden`),         // GET /api/kunden
          fetch(`${baseUrl}/abos`),           // GET /api/abos
          fetch(`${baseUrl}/ermaessigungen`)  // GET /api/ermaessigungen (tbl_Ermaessigung)
        ])

        // .ok prüft ob der HTTP-Statuscode 200–299 ist (= Erfolg)
        if (resKunden.ok) {
          this.kunden = await resKunden.json() // JSON-Text → JavaScript-Array umwandeln
        }
        if (resAbos.ok) {
          this.abos = await resAbos.json()
        }
        if (resErmaessigungen.ok) {
          this.ermaessigungen = await resErmaessigungen.json()
        }
      } catch (err) {
        // Fehler z.B. wenn die API offline ist → Testdaten bleiben erhalten
        console.warn('API ist offline. Nutze lokale Testdaten.', err)
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
</style>
