<template>
  <!-- ==========================================
    AbrechnungsUebersicht.vue
    SOLL-KRITERIUM: Monats- & Jahresübersicht, CSV/PDF-Export
    Daten kommen von C# API: GET /api/abrechnungen?monat=YYYY-MM
    ========================================== -->
  <div class="page">

    <div class="page-header">
      <div>
        <h2 class="page-title">Abrechnungsübersicht</h2>
        <p class="page-desc">Monats- und Jahresübersichten der Einnahmen</p>
      </div>
      <div class="flex gap-sm">
        <!-- JavaScript: @click-Binding ruft exportCSV() auf -->
        <button v-if="ansicht !== 'kunden'" class="btn btn-ghost" id="btn-export-csv" @click="exportCSV">⬇ CSV Export</button>
        <button class="btn btn-ghost" id="btn-export-pdf" @click="exportPDF">⬇ PDF Export</button>
      </div>
    </div>

    <!-- Tab-Navigation -->
    <!-- JavaScript: :class="{ active: ansicht === 'monat' }" setzt aktive CSS-Klasse dynamisch -->
    <div class="tabs" id="abrechnung-tabs">
      <button class="tab-btn" :class="{ active: ansicht === 'monat' }" id="tab-monat"
              @click="ansicht = 'monat'">📅 Monatsübersicht</button>
      <button class="tab-btn" :class="{ active: ansicht === 'jahr' }" id="tab-jahr"
              @click="ansicht = 'jahr'">📆 Jahresübersicht</button>
      <button class="tab-btn" :class="{ active: ansicht === 'kunden' }" id="tab-kunden"
              @click="ansicht = 'kunden'">👤 Kundenabrechnung</button>
    </div>

    <!-- ══════════ MONATSÜBERSICHT ══════════ -->
    <!-- JavaScript: v-if rendert den Block nur wenn ansicht === 'monat' -->
    <div v-if="ansicht === 'monat'" id="view-monat">
      <div class="filter-bar">
        <div class="form-group" style="min-width:200px;">
          <label for="select-monat">Monat wählen</label>
          <!-- JavaScript: v-model bindet gewaehltesMonat an den Monats-Selektor (YYYY-MM) -->
          <input id="select-monat" type="month" v-model="gewaehltesMonat" />
        </div>
        <button class="btn btn-primary" id="btn-monat-laden" @click="ladeMonat">Übersicht laden</button>
      </div>

      <div class="stats-grid stats-grid-3">
        <div class="stat-card">
          <div class="stat-icon green">💶</div>
          <!-- JavaScript: computed 'monatsEinnahmen' summiert alle Endbeträge -->
          <div class="stat-value">{{ formatEuro(monatsEinnahmen) }}</div>
          <div class="stat-label">Einnahmen {{ monatLabel }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon blue">👥</div>
          <!-- JavaScript: .length des reaktiven Arrays -->
          <div class="stat-value">{{ monatsAbrechnungen.length }}</div>
          <div class="stat-label">Abrechnungen</div>
        </div>
        <div class="stat-card">
          <div class="stat-icon orange">🏷</div>
          <div class="stat-value">{{ formatEuro(monatsRabatte) }}</div>
          <div class="stat-label">Ermäßigungen gesamt</div>
        </div>
      </div>

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
                <th>AbrNr</th><th>Kunde</th><th>Abo</th>
                <th>Grundpreis</th><th>Ermäßigung</th><th>Endbetrag</th>
              </tr>
            </thead>
            <tbody>
              <!-- JavaScript: v-for über monatsAbrechnungen-Array (von API geladen) -->
              <tr v-for="abr in monatsAbrechnungen" :key="abr.abrechnungsNr">
                <td class="td-mono">{{ abr.abrechnungsNr }}</td>
                <td>
                  <div class="font-bold">{{ abr.kundenName }}</div>
                  <div class="td-muted">#{{ abr.kundenNr }}</div>
                </td>
                <td><span class="badge badge-primary">{{ abr.aboBezeichnung }}</span></td>
                <td>{{ formatEuro(abr.grundpreis) }}</td>
                <td>
                  <!-- JavaScript: v-if prüft ob Ermäßigung > 0 -->
                  <span v-if="abr.ermaessigung > 0" class="badge badge-warning">-{{ abr.ermaessigung }}%</span>
                  <span v-else class="td-muted">—</span>
                </td>
                <td style="color:var(--clr-accent);font-weight:700;font-size:15px;">
                  {{ formatEuro(abr.endbetrag) }}
                </td>
              </tr>
              <tr v-if="monatsAbrechnungen.length === 0">
                <td colspan="6">
                  <div class="empty-state">
                    <div class="empty-state-icon">📅</div>
                    <h3>Keine Daten für diesen Monat</h3>
                    <p>Wähle einen Monat und lade die Übersicht.</p>
                  </div>
                </td>
              </tr>
            </tbody>
            <!-- JavaScript: v-if zeigt Summenzeile nur wenn Daten vorhanden -->
            <tfoot v-if="monatsAbrechnungen.length > 0">
              <tr style="border-top:2px solid var(--clr-border);">
                <td colspan="5" class="font-bold" style="padding:12px 14px;color:var(--clr-text-muted);">
                  Summe ({{ monatsAbrechnungen.length }} Abrechnungen)
                </td>
                <td style="padding:12px 14px;font-size:17px;font-weight:800;color:var(--clr-accent);">
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
      <div class="filter-bar">
        <div class="form-group" style="min-width:160px;">
          <label for="select-jahr">Jahr wählen</label>
          <!-- JavaScript: v-model.number konvertiert den String-Wert der Option zu Number -->
          <select id="select-jahr" v-model.number="gewaehltesJahr">
            <option v-for="j in verfuegbareJahre" :key="j" :value="j">{{ j }}</option>
          </select>
        </div>
        <button class="btn btn-primary" id="btn-jahr-laden" @click="ladeJahr">Jahresübersicht laden</button>
      </div>

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

      <div class="card">
        <div class="card-header">
          <div class="card-title">Monatliche Einnahmen {{ gewaehltesJahr }}</div>
        </div>

        <!-- Balkendiagramm -->
        <div class="jahres-chart" id="jahres-chart">
          <!-- JavaScript: v-for über 12 Monatsobjekte; :style bindet Höhe dynamisch -->
          <div v-for="(monat, idx) in jahresMonatsDaten" :key="idx"
               class="chart-bar-wrap" :id="'chart-bar-' + idx">
            <div class="chart-bar-label-top">{{ formatEuro(monat.summe) }}</div>
            <div class="chart-bar-outer">
              <!-- JavaScript: :style berechnet Balkenhöhe über balkenHoehe() -->
              <div class="chart-bar-fill" :style="{ height: balkenHoehe(monat.summe) + '%' }"
                   :title="monat.name + ': ' + formatEuro(monat.summe)"></div>
            </div>
            <div class="chart-bar-label">{{ monat.kurzname }}</div>
          </div>
        </div>

        <div class="table-wrapper mt-lg">
          <table>
            <thead>
              <tr><th>Monat</th><th>Abrechnungen</th><th>Einnahmen</th><th>Ermäßigungen</th><th>Netto</th><th>Anteil</th></tr>
            </thead>
            <tbody>
              <tr v-for="(monat, idx) in jahresMonatsDaten" :key="'jt-' + idx">
                <td class="font-bold">{{ monat.name }}</td>
                <td class="td-muted">{{ monat.anzahl }}</td>
                <td style="color:var(--clr-accent);font-weight:600;">{{ formatEuro(monat.summe) }}</td>
                <td style="color:var(--clr-warning);">-{{ formatEuro(monat.rabatte) }}</td>
                <td class="font-bold">{{ formatEuro(monat.netto) }}</td>
                <td>
                  <div class="flex items-center gap-sm">
                    <div class="progress-bar" style="width:80px;">
                      <!-- JavaScript: :style berechnet Breite in % für Fortschrittsbalken -->
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
                <td style="padding:12px 14px;font-size:16px;font-weight:800;color:var(--clr-accent);">{{ formatEuro(jahresEinnahmen) }}</td>
                <td colspan="3"></td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>
    </div>

    <!-- ══════════ KUNDENABRECHNUNG ══════════ -->
    <div v-if="ansicht === 'kunden'" id="view-kundenabrechnung">
      <div class="filter-bar">
        <div class="form-group" style="min-width:240px;">
          <label for="select-abr-kunde">Kunden wählen</label>
          <!-- JavaScript: v-model bindet die gewählte kundennr (int) -->
          <select id="select-abr-kunde" v-model.number="gewaehltesKundeNr">
            <option :value="0">– Kunden wählen –</option>
            <!-- JavaScript: v-for erzeugt Optionen aus dem kunden-Array (camelCase-Felder!) -->
            <option v-for="k in kunden" :key="k.kundennr" :value="k.kundennr">
              {{ k.nachname }}, {{ k.vorname }} (#{{ k.kundennr }})
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
      <!-- JavaScript: v-if zeigt Karte nur wenn ein Kunde ausgewählt ist (kundennr > 0) -->
      <div class="card" v-if="gewaehltesKundeNr > 0">
        <div class="card-header">
          <div>
            <div class="card-title">Abrechnung für {{ kundeDetail?.nachname }}, {{ kundeDetail?.vorname }}</div>
            <div class="card-subtitle">
              KundenNr: {{ gewaehltesKundeNr }} |
              IBAN: {{ ibanMask(kundeDetail?.iban || kundeDetail?.konto?.iban) }}
            </div>
          </div>
          <div style="text-align:right;">
            <div style="font-size:28px;font-weight:800;color:var(--clr-accent);">
              <!-- JavaScript: computed 'kundeMonatsbeitrag' berechnet Preis mit Ermäßigung -->
              {{ formatEuro(kundeMonatsbeitrag) }}
            </div>
            <div class="text-muted">aktueller Monatsbeitrag (geschätzt)</div>
          </div>
        </div>

        <hr class="divider" />

        <div class="preisaufschluesselung">
          <div class="preis-zeile">
            <span>Grundpreis</span>
            <span>{{ formatEuro(kundeGrundpreis) }}</span>
          </div>
          <!-- JavaScript: v-if zeigt Rabattzeile nur wenn Ermäßigung vorhanden -->
          <div class="preis-zeile" v-if="kundeErmaessigung > 0" style="color:var(--clr-warning);">
            <span>Ermäßigung ({{ (kundeErmaessigung * 100).toFixed(0) }}%)</span>
            <span>-{{ formatEuro(kundeRabattBetrag) }}</span>
          </div>
          <hr class="divider" />
          <div class="preis-zeile font-bold" style="font-size:16px;">
            <span>Zu zahlender Betrag</span>
            <span style="color:var(--clr-accent);">{{ formatEuro(kundeMonatsbeitrag) }}</span>
          </div>
        </div>
      </div>

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
   ██ JAVASCRIPT – AbrechnungsUebersicht.vue
   Zeigt Monats-/Jahresabrechnungen und Kundenabrechnung.
   API-Endpunkte:
     GET /api/abrechnungen?monat=2026-05  → Monatsübersicht
     GET /api/abrechnungen?jahr=2026      → Jahresübersicht (12 Objekte)
   Props: kunden[], abos[], ermaessigungen[] (camelCase!)
   ========================================================= */

export default {
  name: 'AbrechnungsUebersicht',

  props: {
    kunden:         { type: Array, default: () => [] },
    abos:           { type: Array, default: () => [] },
    ermaessigungen: { type: Array, default: () => [] }
  },

  data() {
    const heute = new Date()
    // Aktuellen Monat als YYYY-MM vorausfüllen
    const monatStr = `${heute.getFullYear()}-${String(heute.getMonth()+1).padStart(2,'0')}`
    return {
      ansicht:            'monat',  // 'monat' | 'jahr' | 'kunden'
      gewaehltesMonat:    monatStr,
      monatsAbrechnungen: [],       // Wird von ladeMonat() befüllt
      gewaehltesJahr:     heute.getFullYear(),
      jahresMonatsDaten:  this.emptyJahresDaten(), // 12 leere Monatsobjekte
      gewaehltesKundeNr:  0,        // kundennr (int) des gewählten Kunden
      kundenAbrZeitraum:  monatStr
    }
  },

  computed: {
    // Lesbare Monatsbezeichnung aus "YYYY-MM" → "Mai 2026"
    monatLabel() {
      if (!this.gewaehltesMonat) return ''
      const [y, m] = this.gewaehltesMonat.split('-')
      const namen = ['','Januar','Februar','März','April','Mai','Juni',
                     'Juli','August','September','Oktober','November','Dezember']
      return `${namen[parseInt(m)]} ${y}`
    },

    // Summe aller Endbeträge des gewählten Monats
    monatsEinnahmen() {
      return this.monatsAbrechnungen.reduce((s, a) => s + (parseFloat(a.endbetrag) || 0), 0)
    },
    // Summe aller Rabatte (Differenz Grundpreis - Endbetrag)
    monatsRabatte() {
      return this.monatsAbrechnungen.reduce((s, a) => {
        return s + Math.max(0, (parseFloat(a.grundpreis) || 0) - (parseFloat(a.endbetrag) || 0))
      }, 0)
    },

    // Jahresstatistiken
    jahresEinnahmen()    { return this.jahresMonatsDaten.reduce((s, m) => s + (m.netto || 0), 0) },
    durchschnittMonat()  { return this.jahresEinnahmen / 12 },
    besterMonatName()    {
      if (!this.jahresMonatsDaten.length) return '—'
      return this.jahresMonatsDaten.reduce((a, b) => (a.netto || 0) >= (b.netto || 0) ? a : b).name
    },
    schwächsterMonatName() {
      if (!this.jahresMonatsDaten.length) return '—'
      return this.jahresMonatsDaten.reduce((a, b) => (a.netto || 0) <= (b.netto || 0) ? a : b).name
    },
    gesamtAbrechnungen() { return this.jahresMonatsDaten.reduce((s, m) => s + (m.anzahl || 0), 0) },

    // Auswählbare Jahre (aktuelles Jahr ± 2)
    verfuegbareJahre() {
      const j = new Date().getFullYear()
      return [j-2, j-1, j, j+1]
    },

    // ── Kundenabrechnung (computed aus Props) ────────────────────────

    // Objekt des gewählten Kunden
    kundeDetail() {
      return this.kunden.find(k => k.kundennr === this.gewaehltesKundeNr) || null
    },
    // Grundpreis des letzten/aktiven Abos des Kunden
    kundeGrundpreis() {
      const activeAbo = this.kundeDetail?.aktivesAbo?.abo
      return activeAbo ? parseFloat(activeAbo.grundpreis || 0) : 0
    },
    // Ermäßigungssatz des Kunden (aus tbl_Abrechnung/tbl_Ermaessigte)
    kundeErmaessigung() {
      const erm = this.kundeDetail?.ermaessigung
      return erm ? parseFloat(erm.ermaessigungssatz || 0) : 0
    },
    // Rabattbetrag in Euro
    kundeRabattBetrag() {
      return this.kundeGrundpreis * this.kundeErmaessigung
    },
    // Monatlicher Endbetrag nach Ermäßigung
    kundeMonatsbeitrag() {
      return this.kundeGrundpreis * (1 - this.kundeErmaessigung)
    }
  },

  methods: {
    /* ── ladeMonat() ──────────────────────────────────────────────────
       Lädt Abrechnungen für den gewählten Monat von der C# API.
       API liefert: [{ abrechnungsNr, kundenName, kundenNr, aboBezeichnung,
                       grundpreis, ermaessigung, endbetrag, sonderangebot }]
    ──────────────────────────────────────────────────────────────── */
    async ladeMonat() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        const res = await fetch(`${baseUrl}/abrechnungen?monat=${this.gewaehltesMonat}`)
        if (res.ok) {
          this.monatsAbrechnungen = await res.json()
        } else {
          throw new Error(`API Status: ${res.status}`)
        }
      } catch (err) {
        console.warn('API offline – Testdaten für Monatsübersicht.', err)
        // Fallback: Abrechnungen aus Kunden und Abos berechnen
        this.monatsAbrechnungen = this.kunden.map((k, idx) => ({
          abrechnungsNr:  idx + 1,
          kundenName:     `${k.vorname} ${k.nachname}`,
          kundenNr:       k.kundennr,
          aboBezeichnung: this.abos[0] ? `Abo #${this.abos[0].abonr}` : 'Standard',
          grundpreis:     this.abos[0]?.grundpreis || 0,
          ermaessigung:   0,
          endbetrag:      this.abos[0]?.grundpreis || 0,
          sonderangebot:  null
        }))
      }
    },

    /* ── ladeJahr() ───────────────────────────────────────────────────
       Lädt Jahresabrechnungen von der C# API.
       API liefert: [{ name, kurzname, anzahl, summe, rabatte, netto }] (12 Einträge)
    ──────────────────────────────────────────────────────────────── */
    async ladeJahr() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        const res = await fetch(`${baseUrl}/abrechnungen?jahr=${this.gewaehltesJahr}`)
        if (res.ok) {
          this.jahresMonatsDaten = await res.json()
        } else {
          throw new Error(`API Status: ${res.status}`)
        }
      } catch (err) {
        console.warn('API offline – generierte Jahresübersicht.', err)
        // Fallback: plausible Testdaten generieren
        const basis = this.kunden.length * (this.abos[0]?.grundpreis || 0)
        const namen = ['Januar','Februar','März','April','Mai','Juni','Juli','August','September','Oktober','November','Dezember']
        const kurz  = ['Jan','Feb','Mär','Apr','Mai','Jun','Jul','Aug','Sep','Okt','Nov','Dez']
        this.jahresMonatsDaten = namen.map((name, i) => {
          const f = 0.85 + Math.sin(i) * 0.15
          const summe = basis * f
          return { name, kurzname: kurz[i], anzahl: Math.round(this.kunden.length * f),
                   summe: parseFloat(summe.toFixed(2)), rabatte: 0, netto: parseFloat(summe.toFixed(2)) }
        })
      }
    },

    // Wird aufgerufen wenn Kunde oder Zeitraum im Kunden-Tab geändert wird
    async ladeKundenabrechnung() {
      // Die Berechnung läuft über computed Properties
      console.log('Kundenabrechnung wird aus API-Daten berechnet.')
    },

    // CSV-Export der aktuell angezeigten Daten
    // CSV-Export der aktuell angezeigten Daten
    exportCSV() {
      let zeilen = []
      if (this.ansicht === 'monat') {
        zeilen.push('AbrNr;Kunde;KundenNr;Abo;Grundpreis;Ermäßigung;Endbetrag')
        this.monatsAbrechnungen.forEach(a => {
          zeilen.push(`${a.abrechnungsNr};${a.kundenName};${a.kundenNr};${a.aboBezeichnung};${a.grundpreis};${a.ermaessigung}%;${a.endbetrag}`)
        })
      } else if (this.ansicht === 'jahr') {
        zeilen.push('Monat;Abrechnungen;Einnahmen;Ermäßigungen;Netto')
        this.jahresMonatsDaten.forEach(m => {
          zeilen.push(`${m.name};${m.anzahl};${m.summe};${m.rabatte};${m.netto}`)
        })
      }
      // Browser-Download ohne Server erzeugen
      const blob = new Blob([zeilen.join('\n')], { type: 'text/csv;charset=utf-8;' })
      const url  = URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href     = url
      link.download = `abrechnung_${this.ansicht}_${new Date().toISOString().slice(0,10)}.csv`
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      URL.revokeObjectURL(url)
    },

    // PDF-Export über den Browser-Druckdialog
    exportPDF() { window.print() },

    // Formatiert Zahl als Euro-Betrag (49.9 → "49,90 €")
    formatEuro(v) {
      return parseFloat(v || 0).toFixed(2).replace('.', ',') + ' €'
    },

    // Berechnet relative Balkenhöhe für das Diagramm (0-100%)
    balkenHoehe(summe) {
      const max = Math.max(...this.jahresMonatsDaten.map(m => m.summe), 1)
      return Math.round((summe / max) * 100)
    },

    // Prozentualer Anteil eines Monats an den Jahreseinnahmen
    anteil(summe) {
      if (!this.jahresEinnahmen) return 0
      return Math.round((summe / this.jahresEinnahmen) * 100)
    },

    // Erstellt 12 leere Monatsobjekte als Startzustand
    emptyJahresDaten() {
      const namen = ['Januar','Februar','März','April','Mai','Juni','Juli','August','September','Oktober','November','Dezember']
      const kurz  = ['Jan','Feb','Mär','Apr','Mai','Jun','Jul','Aug','Sep','Okt','Nov','Dez']
      return namen.map((name, i) => ({ name, kurzname: kurz[i], anzahl: 0, summe: 0, rabatte: 0, netto: 0 }))
    },

    // IBAN maskieren: DE89370400440532013000 → "DE89 **** **** **** 3000"
    ibanMask(iban) {
      if (!iban) return '—'
      const clean = iban.replace(/\s/g, '')
      return clean.slice(0, 4) + ' **** **** **** ' + clean.slice(-4)
    }
  },

  created() {
    this.jahresMonatsDaten = this.emptyJahresDaten()
  },

  mounted() {
    this.ladeMonat()
    this.ladeJahr()
  }
}
</script>

<style scoped>
.jahres-chart { display:flex; align-items:flex-end; gap:8px; height:200px; padding:16px 0 8px; overflow-x:auto; }
.chart-bar-wrap { flex:1; min-width:36px; display:flex; flex-direction:column; align-items:center; height:100%; position:relative; }
.chart-bar-label-top { font-size:9px; color:var(--clr-text-dim); margin-bottom:4px; white-space:nowrap; }
.chart-bar-outer { flex:1; width:100%; display:flex; align-items:flex-end; background:var(--clr-bg); border-radius:4px 4px 0 0; overflow:hidden; border:1px solid var(--clr-border); }
.chart-bar-fill { width:100%; background:linear-gradient(180deg, var(--clr-primary), var(--clr-accent)); border-radius:4px 4px 0 0; transition:height 600ms ease; min-height:4px; }
.chart-bar-label { font-size:10px; color:var(--clr-text-dim); margin-top:4px; font-weight:600; }
.preisaufschluesselung { display:flex; flex-direction:column; gap:var(--sp-sm); max-width:420px; }
.preis-zeile { display:flex; justify-content:space-between; font-size:14px; padding:4px 0; }
@media (min-width:1280px) { .stats-grid-3 { grid-template-columns:repeat(3,1fr) !important; } }
</style>
