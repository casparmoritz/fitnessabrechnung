<template>
  <!-- ==========================================
    AbrechnungsUebersicht.vue
    SOLL-KRITERIUM: Monats- & Jahresübersicht, Export (CSV/PDF)
    ========================================== -->
  <div class="page">

    <div class="page-header">
      <div>
        <h2 class="page-title">Abrechnungsübersicht</h2>
        <p class="page-desc">Monats- und Jahresübersichten der Einnahmen</p>
      </div>
      <div class="flex gap-sm">
        <button class="btn btn-ghost" id="btn-export-csv" @click="exportCSV">
          ⬇ CSV Export
        </button>
        <button class="btn btn-ghost" id="btn-export-pdf" @click="exportPDF">
          ⬇ PDF Export
        </button>
      </div>
    </div>

    <!-- Tab: Monats / Jahres -->
    <div class="tabs" id="abrechnung-tabs">
      <button class="tab-btn" :class="{ active: ansicht === 'monat' }" id="tab-monat" @click="ansicht = 'monat'">
        📅 Monatsübersicht
      </button>
      <button class="tab-btn" :class="{ active: ansicht === 'jahr' }" id="tab-jahr" @click="ansicht = 'jahr'">
        📆 Jahresübersicht
      </button>
      <button class="tab-btn" :class="{ active: ansicht === 'kunden' }" id="tab-kunden" @click="ansicht = 'kunden'">
        👤 Kundenabrechnung
      </button>
    </div>

    <!-- ══════════ MONATSÜBERSICHT ══════════ -->
    <div v-if="ansicht === 'monat'" id="view-monat">

      <!-- Filter: Monat + Jahr -->
      <div class="filter-bar">
        <div class="form-group" style="min-width:200px;">
          <label for="select-monat">Monat wählen</label>
          <input id="select-monat" type="month" v-model="gewaehltesMonat" />
        </div>
        <button class="btn btn-primary" id="btn-monat-laden" @click="ladeMonat">
          Übersicht laden
        </button>
      </div>

      <!-- Stat-Kacheln Monat -->
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-icon green">💶</div>
          <div class="stat-value">{{ formatEuro(monatsEinnahmen) }}</div>
          <div class="stat-label">Einnahmen {{ monatLabel }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon blue">👥</div>
          <div class="stat-value">{{ monatsAbrechnungen.length }}</div>
          <div class="stat-label">Abrechnungen</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon orange">🏷</div>
          <div class="stat-value">{{ formatEuro(monatsRabatte) }}</div>
          <div class="stat-label">Ermäßigungen gesamt</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon teal">📈</div>
          <div class="stat-value">{{ formatEuro(monatsNetto) }}</div>
          <div class="stat-label">Netto-Einnahmen</div>
        </div>
      </div>

      <!-- Tabelle Monatsabrechnungen -->
      <div class="card">
        <div class="card-header">
          <div>
            <div class="card-title">Abrechnungen – {{ monatLabel }}</div>
            <div class="card-subtitle">Alle Kundenabrechnungen des gewählten Monats</div>
          </div>
        </div>
        <div class="table-wrapper">
          <table>
            <thead>
              <tr>
                <th>AbrNr</th>
                <th>Kunde</th>
                <th>Abo</th>
                <th>Grundpreis</th>
                <th>Ermäßigung</th>
                <th>Sonderangebot</th>
                <th>Endbetrag</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="abr in monatsAbrechnungen" :key="abr.abrechnungsNr" :id="'row-abr-' + abr.abrechnungsNr">
                <td class="td-mono">{{ abr.abrechnungsNr }}</td>
                <td>
                  <div class="font-bold">{{ abr.kundenName }}</div>
                  <div class="td-muted">#{{ abr.kundenNr }}</div>
                </td>
                <td>
                  <span class="badge badge-primary">{{ abr.aboBezeichnung }}</span>
                </td>
                <td>{{ formatEuro(abr.grundpreis) }}</td>
                <td>
                  <span v-if="abr.ermaessigung > 0" class="badge badge-warning">
                    -{{ abr.ermaessigung }}%
                  </span>
                  <span v-else class="td-muted">—</span>
                </td>
                <td>
                  <span v-if="abr.sonderangebot" class="badge badge-accent">{{ abr.sonderangebot }}</span>
                  <span v-else class="td-muted">—</span>
                </td>
                <td style="color:var(--clr-accent); font-weight:700; font-size:15px;">
                  {{ formatEuro(abr.endbetrag) }}
                </td>
              </tr>
              <tr v-if="monatsAbrechnungen.length === 0">
                <td colspan="7">
                  <div class="empty-state">
                    <div class="empty-state-icon">📅</div>
                    <h3>Keine Daten für diesen Monat</h3>
                    <p>Wähle einen Monat und lade die Übersicht.</p>
                  </div>
                </td>
              </tr>
            </tbody>
            <!-- Summenzeile -->
            <tfoot v-if="monatsAbrechnungen.length > 0">
              <tr style="border-top: 2px solid var(--clr-border);">
                <td colspan="6" class="font-bold" style="padding: 12px 14px; color:var(--clr-text-muted);">
                  Summe ({{ monatsAbrechnungen.length }} Abrechnungen)
                </td>
                <td style="padding:12px 14px; font-size:17px; font-weight:800; color:var(--clr-accent);">
                  {{ formatEuro(monatsEinnahmen) }}
                </td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>
    </div>

    <!-- ══════════ JAHRESÜBERSICHT ══════════ -->
    <div v-if="ansicht === 'jahr'" id="view-jahr">

      <!-- Filter: Jahr -->
      <div class="filter-bar">
        <div class="form-group" style="min-width:160px;">
          <label for="select-jahr">Jahr wählen</label>
          <select id="select-jahr" v-model.number="gewaehltesJahr">
            <option v-for="j in verfuegbareJahre" :key="j" :value="j">{{ j }}</option>
          </select>
        </div>
        <button class="btn btn-primary" id="btn-jahr-laden" @click="ladeJahr">
          Jahresübersicht laden
        </button>
      </div>

      <!-- Jahres-Stat-Kacheln -->
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-icon green">💶</div>
          <div class="stat-value">{{ formatEuro(jahresEinnahmen) }}</div>
          <div class="stat-label">Jahreseinnahmen {{ gewaehltesJahr }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon blue">📊</div>
          <div class="stat-value">{{ formatEuro(durchschnittMonat) }}</div>
          <div class="stat-label">Ø pro Monat</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon teal">📈</div>
          <div class="stat-value">{{ besterMonatName }}</div>
          <div class="stat-label">Stärkster Monat</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon orange">📉</div>
          <div class="stat-value">{{ schwächsterMonatName }}</div>
          <div class="stat-label">Schwächster Monat</div>
        </div>
      </div>

      <!-- Monatsbalken -->
      <div class="card">
        <div class="card-header">
          <div class="card-title">Monatliche Einnahmen {{ gewaehltesJahr }}</div>
        </div>

        <div class="jahres-chart" id="jahres-chart">
          <div
            v-for="(monat, idx) in jahresMonatsDaten"
            :key="idx"
            class="chart-bar-wrap"
            :id="'chart-bar-' + idx"
          >
            <div class="chart-bar-label-top">{{ formatEuro(monat.summe) }}</div>
            <div class="chart-bar-outer">
              <div
                class="chart-bar-fill"
                :style="{ height: balkenHoehe(monat.summe) + '%' }"
                :title="monat.name + ': ' + formatEuro(monat.summe)"
              ></div>
            </div>
            <div class="chart-bar-label">{{ monat.kurzname }}</div>
          </div>
        </div>

        <!-- Jahrestabelle -->
        <div class="table-wrapper mt-lg">
          <table>
            <thead>
              <tr>
                <th>Monat</th>
                <th>Abrechnungen</th>
                <th>Einnahmen</th>
                <th>Ermäßigungen</th>
                <th>Netto</th>
                <th>Anteil</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(monat, idx) in jahresMonatsDaten" :key="'jt-' + idx">
                <td class="font-bold">{{ monat.name }}</td>
                <td class="td-muted">{{ monat.anzahl }}</td>
                <td style="color:var(--clr-accent); font-weight:600;">{{ formatEuro(monat.summe) }}</td>
                <td style="color:var(--clr-warning);">-{{ formatEuro(monat.rabatte) }}</td>
                <td class="font-bold">{{ formatEuro(monat.netto) }}</td>
                <td>
                  <div class="flex items-center gap-sm">
                    <div class="progress-bar" style="width:80px;">
                      <div class="progress-fill" :style="{ width: anteil(monat.summe) + '%' }"></div>
                    </div>
                    <span class="text-muted">{{ anteil(monat.summe) }}%</span>
                  </div>
                </td>
              </tr>
            </tbody>
            <tfoot>
              <tr style="border-top:2px solid var(--clr-border);">
                <td class="font-bold" style="padding:12px 14px;">Gesamt {{ gewaehltesJahr }}</td>
                <td class="font-bold" style="padding:12px 14px;">{{ gesamtAbrechnungen }}</td>
                <td style="padding:12px 14px; font-size:16px; font-weight:800; color:var(--clr-accent);">{{ formatEuro(jahresEinnahmen) }}</td>
                <td colspan="3"></td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>
    </div>

    <!-- ══════════ KUNDENABRECHNUNG ══════════ -->
    <div v-if="ansicht === 'kunden'" id="view-kundenabrechnung">

      <!-- Kunden-Select -->
      <div class="filter-bar">
        <div class="form-group" style="min-width:240px;">
          <label for="select-abr-kunde">Kunden wählen</label>
          <select id="select-abr-kunde" v-model="gewaehltesKunde">
            <option value="">– Kunden wählen –</option>
            <option v-for="k in kunden" :key="k.kundenNr" :value="k.kundenNr">
              {{ k.nachname }}, {{ k.vorname }} (#{{ k.kundenNr }})
            </option>
          </select>
        </div>
        <div class="form-group" style="min-width:200px;">
          <label for="abr-kunde-monat">Zeitraum</label>
          <input id="abr-kunde-monat" type="month" v-model="kundenAbrZeitraum" />
        </div>
        <button class="btn btn-primary" id="btn-kundabr-laden" @click="ladeKundenabrechnung">Laden</button>
      </div>

      <!-- Kunden-Detail-Karte -->
      <div class="card" v-if="gewaehltesKunde">
        <div class="card-header">
          <div>
            <div class="card-title">Abrechnung für {{ kundeDetail?.nachname }}, {{ kundeDetail?.vorname }}</div>
            <div class="card-subtitle">KundenNr: {{ gewaehltesKunde }} · Abo: {{ kundeAboName }}</div>
          </div>
          <div style="text-align:right;">
            <div style="font-size:28px; font-weight:800; color:var(--clr-accent);">
              {{ formatEuro(kundeMonatsbeitrag) }}
            </div>
            <div class="text-muted">aktueller Monatsbeitrag</div>
          </div>
        </div>

        <hr class="divider" />

        <!-- Preisberechnung Aufschlüsselung -->
        <div class="preisaufschluesselung">
          <div class="preis-zeile">
            <span>Grundpreis ({{ kundeAboName }})</span>
            <span>{{ formatEuro(kundeGrundpreis) }}</span>
          </div>
          <div class="preis-zeile" v-if="kundeErmaessigung > 0" style="color:var(--clr-warning);">
            <span>Ermäßigung ({{ kundeErmaessigung }}%)</span>
            <span>-{{ formatEuro(kundeRabattBetrag) }}</span>
          </div>
          <div class="preis-zeile" v-if="kundeSonderangebot" style="color:var(--clr-accent);">
            <span>Sonderangebot ({{ kundeSonderangebot }})</span>
            <span>inklusive</span>
          </div>
          <hr class="divider" />
          <div class="preis-zeile font-bold" style="font-size:16px;">
            <span>Zu zahlender Betrag</span>
            <span style="color:var(--clr-accent);">{{ formatEuro(kundeMonatsbeitrag) }}</span>
          </div>
        </div>
      </div>

      <!-- Empty wenn kein Kunde gewählt -->
      <div class="card" v-else>
        <div class="empty-state">
          <div class="empty-state-icon">👤</div>
          <h3>Kein Kunde ausgewählt</h3>
          <p>Wähle einen Kunden aus der Liste, um die individuelle Abrechnung zu sehen.</p>
        </div>
      </div>

    </div>

  </div>
</template>

<script>
/* =========================================================
   JAVASCRIPT – AbrechnungsUebersicht.vue
   Monats-/Jahresübersicht und Kundenabrechnung
   Props:  kunden[], abos[], ermaessigungen[] (aus App.vue)
   ========================================================= */

export default {
  name: 'AbrechnungsUebersicht',

  // Props = Daten die von App.vue übergeben werden
  props: {
    kunden:         { type: Array, default: () => [] }, // Alle Kunden (für Dropdown)
    abos:           { type: Array, default: () => [] }, // Alle Abos (für Preisberechnung)
    ermaessigungen: { type: Array, default: () => [] }  // Alle Ermäßigungen
  },

  // data() = alle reaktiven Variablen dieser Komponente
  data() {
    const heute = new Date()
    return {
      // Welcher Tab aktiv ist: 'monat', 'jahr' oder 'kunden'
      ansicht: 'monat',

      // Gewählter Monat im Format YYYY-MM (vorbelegt mit aktuellem Monat)
      gewaehltesMonat: `${heute.getFullYear()}-${String(heute.getMonth()+1).padStart(2,'0')}`,
      // Die von der API geladenen Abrechnungen für den gewählten Monat
      monatsAbrechnungen: [],

      // Gewähltes Jahr (vorbelegt mit aktuellem Jahr)
      gewaehltesJahr: heute.getFullYear(),
      // 12 Monatsobjekte mit { name, kurzname, anzahl, summe, rabatte, netto }
      jahresMonatsDaten: this.emptyJahresDaten(),

      // Gewählter Kunde im Kunden-Tab (KundenNr als String)
      gewaehltesKunde: '',
      // Zeitraum für die Kundenabrechnung (Format YYYY-MM)
      kundenAbrZeitraum: `${heute.getFullYear()}-${String(heute.getMonth()+1).padStart(2,'0')}`
    }
  },

  // computed = alle berechneten Werte für die verschiedenen Tabs und Statistiken
  computed: {
    // Gibt den Monatsnamen als lesbaren Text zurück (z.B. "2026-05" → "Mai 2026")
    monatLabel() {
      if (!this.gewaehltesMonat) return ''
      const [y,m] = this.gewaehltesMonat.split('-')
      const namen = ['','Januar','Februar','März','April','Mai','Juni','Juli','August','September','Oktober','November','Dezember']
      return `${namen[parseInt(m)]} ${y}`
    },

    // Summe aller Endbeträge der Monatsabrechnungen
    monatsEinnahmen() {
      return this.monatsAbrechnungen.reduce((s, a) => s + (parseFloat(a.endbetrag)||0), 0)
    },
    // Summe aller Rabatte (Grundpreis - Endbetrag) des Monats
    monatsRabatte() {
      return this.monatsAbrechnungen.reduce((s, a) => {
        const basis = parseFloat(a.grundpreis)||0
        const end   = parseFloat(a.endbetrag)||0
        return s + Math.max(0, basis - end)
      }, 0)
    },
    // Netto = Einnahmen minus Rabatte
    monatsNetto() { return this.monatsEinnahmen - this.monatsRabatte },

    // Summe aller Monats-Summen des Jahres
    jahresEinnahmen() {
      return this.jahresMonatsDaten.reduce((s, m) => s + (parseFloat(m.summe)||0), 0)
    },
    // Durchschnittliche Monatseinnahmen des Jahres
    durchschnittMonat() {
      return this.jahresEinnahmen / 12
    },

    // Name des Monats mit den höchsten Einnahmen
    besterMonatName() {
      if (!this.jahresMonatsDaten.length) return '—'
      return this.jahresMonatsDaten.reduce((a,b) => a.summe >= b.summe ? a : b).name
    },
    // Name des Monats mit den niedrigsten Einnahmen
    schwächsterMonatName() {
      if (!this.jahresMonatsDaten.length) return '—'
      return this.jahresMonatsDaten.reduce((a,b) => a.summe <= b.summe ? a : b).name
    },
    // Gesamtzahl aller Abrechnungen im Jahr
    gesamtAbrechnungen() {
      return this.jahresMonatsDaten.reduce((s, m) => s + (m.anzahl||0), 0)
    },

    // Array der wählbaren Jahre (2 Jahre davor bis 1 Jahr danach)
    verfuegbareJahre() {
      const j = new Date().getFullYear()
      return [j-2, j-1, j, j+1]
    },

    // Gibt das vollständige Kunden-Objekt des gewählten Kunden zurück
    kundeDetail() {
      return this.kunden.find(k => k.kundenNr === this.gewaehltesKunde) || null
    },
    // Name des Abos des gewählten Kunden
    kundeAboName() {
      const abo = this.abos.find(a => a.aboNr === this.kundeDetail?.aboNr)
      return abo?.bezeichnung || '– kein Abo –'
    },
    // Grundpreis des Abos des gewählten Kunden
    kundeGrundpreis() {
      return parseFloat(this.abos.find(a => a.aboNr === this.kundeDetail?.aboNr)?.grundpreis || 0)
    },
    // Ermäßigungssatz des gewählten Kunden in Prozent
    kundeErmaessigung() {
      return parseFloat(this.kundeDetail?.ermaessigungssatz || 0)
    },
    // Absoluter Rabattbetrag in Euro
    kundeRabattBetrag() {
      return this.kundeGrundpreis * (this.kundeErmaessigung / 100)
    },
    // Sonderangebot-Text des Abos (oder leer)
    kundeSonderangebot() {
      return this.abos.find(a => a.aboNr === this.kundeDetail?.aboNr)?.sonderangebot || ''
    },
    // Finaler Monatsbeitrag des Kunden nach Ermäßigung
    kundeMonatsbeitrag() {
      return this.kundeGrundpreis * (1 - this.kundeErmaessigung / 100)
    }
  },

  methods: {
    // ✅ IMPLEMENTIERT: Monatsabrechnungen von der API laden
    async ladeMonat() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        // Fragt die Abrechnungen für den gewählten Monat ab (Format: YYYY-MM)
        const res = await fetch(`${baseUrl}/abrechnungen?monat=${this.gewaehltesMonat}`)
        if (res.ok) {
          this.monatsAbrechnungen = await res.json()
        }
      } catch (err) {
        console.warn('API offline – keine Monatsdaten geladen.', err)
      }
    },

    // ✅ IMPLEMENTIERT: Jahresabrechnungen von der API laden
    async ladeJahr() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        // API liefert 12 Monatsobjekte: { name, kurzname, anzahl, summe, rabatte, netto }
        const res = await fetch(`${baseUrl}/abrechnungen?jahr=${this.gewaehltesJahr}`)
        if (res.ok) {
          this.jahresMonatsDaten = await res.json()
        }
      } catch (err) {
        console.warn('API offline – keine Jahresdaten geladen.', err)
      }
    },

    // ✅ IMPLEMENTIERT: Kundenabrechnung – Berechnung läuft über computed Properties
    async ladeKundenabrechnung() {
      // Die Anzeige (Preis, Ermäßigung, Abo) wird automatisch über computed berechnet
      // sobald ein Kunde im Dropdown gewählt wird (kundeDetail, kundeMonatsbeitrag etc.)
      // Ein eigener API-Call ist hier nicht notwendig
      console.log('Kundenabrechnung wird aus lokalen Props berechnet.')
    },

    // ✅ IMPLEMENTIERT: Tabellendaten als CSV-Datei herunterladen
    exportCSV() {
      let zeilen = []

      if (this.ansicht === 'monat') {
        // Kopfzeile der CSV-Datei
        zeilen.push('AbrNr;Kunde;KundenNr;Abo;Grundpreis;Ermäßigung;Endbetrag')
        // Eine Zeile pro Abrechnung
        this.monatsAbrechnungen.forEach(a => {
          zeilen.push(
            `${a.abrechnungsNr};${a.kundenName};${a.kundenNr};${a.aboBezeichnung};${a.grundpreis};${a.ermaessigung}%;${a.endbetrag}`
          )
        })
      } else if (this.ansicht === 'jahr') {
        zeilen.push('Monat;Abrechnungen;Einnahmen;Ermäßigungen;Netto')
        this.jahresMonatsDaten.forEach(m => {
          zeilen.push(`${m.name};${m.anzahl};${m.summe};${m.rabatte};${m.netto}`)
        })
      }

      // Browser-Download: Inhalt als Datei anbieten ohne Server
      const inhalt = zeilen.join('\n')
      const blob = new Blob([inhalt], { type: 'text/csv;charset=utf-8;' })
      const url = URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href = url
      link.download = `abrechnung_${this.ansicht}_export.csv`
      link.click()
      URL.revokeObjectURL(url) // Speicher wieder freigeben
    },

    // ✅ IMPLEMENTIERT: Seite als PDF exportieren (öffnet Browser-Druckdialog)
    exportPDF() {
      window.print()
    },

    // Formatiert eine Zahl als Euro-Betrag (z.B. 49.9 → "49,90 €")
    formatEuro(v) {
      return parseFloat(v || 0).toFixed(2).replace('.', ',') + ' €'
    },

    // Berechnet die Balkenhöhe in % relativ zum höchsten Monatswert (für das Diagramm)
    balkenHoehe(summe) {
      const max = Math.max(...this.jahresMonatsDaten.map(m => m.summe), 1)
      return Math.round((summe / max) * 100)
    },

    // Berechnet den prozentualen Anteil eines Monats an den Jahreseinnahmen
    anteil(summe) {
      if (!this.jahresEinnahmen) return 0
      return Math.round((summe / this.jahresEinnahmen) * 100)
    },

    // Erstellt ein leeres Array mit 12 Monatsobjekten (Startzustand für Jahresübersicht)
    emptyJahresDaten() {
      const namen = ['Januar','Februar','März','April','Mai','Juni','Juli','August','September','Oktober','November','Dezember']
      const kurz  = ['Jan','Feb','Mär','Apr','Mai','Jun','Jul','Aug','Sep','Okt','Nov','Dez']
      return namen.map((name, i) => ({
        name, kurzname: kurz[i], anzahl: 0, summe: 0, rabatte: 0, netto: 0
      }))
    }
  },

  // created() läuft bevor das HTML gerendert wird → Jahresdaten vorinitialisieren
  created() {
    this.jahresMonatsDaten = this.emptyJahresDaten()
  }
}
</script>

<style scoped>
/* Balkendiagramm */
.jahres-chart {
  display: flex;
  align-items: flex-end;
  gap: 8px;
  height: 200px;
  padding: 16px 0 8px;
  overflow-x: auto;
}
.chart-bar-wrap {
  flex: 1;
  min-width: 36px;
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  position: relative;
}
.chart-bar-label-top {
  font-size: 9px;
  color: var(--clr-text-dim);
  margin-bottom: 4px;
  white-space: nowrap;
}
.chart-bar-outer {
  flex: 1;
  width: 100%;
  display: flex;
  align-items: flex-end;
  background: var(--clr-bg);
  border-radius: 4px 4px 0 0;
  overflow: hidden;
  border: 1px solid var(--clr-border);
}
.chart-bar-fill {
  width: 100%;
  background: linear-gradient(180deg, var(--clr-primary), var(--clr-accent));
  border-radius: 4px 4px 0 0;
  transition: height 600ms ease;
  min-height: 4px;
}
.chart-bar-label {
  font-size: 10px;
  color: var(--clr-text-dim);
  margin-top: 4px;
  font-weight: 600;
}

/* Preisaufschlüsselung */
.preisaufschluesselung {
  display: flex;
  flex-direction: column;
  gap: var(--sp-sm);
  max-width: 420px;
}
.preis-zeile {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  padding: 4px 0;
}
</style>
