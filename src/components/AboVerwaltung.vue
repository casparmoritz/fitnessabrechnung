<template>
  <!-- ==========================================
    AboVerwaltung.vue
    MUSS-KRITERIUM: Aboverträge anlegen, bearbeiten, löschen
    Feldnamen der C# API: abonr, grundpreis, kuendigsfrist, kurs (int), getraenke (int), laufzeit
    ========================================== -->
  <div class="page">

    <div class="page-header">
      <div>
        <h2 class="page-title">Abo-Verwaltung</h2>
        <p class="page-desc">Abotypen mit Grundpreisen und Zusatzoptionen verwalten</p>
      </div>
      <!-- JavaScript: @click öffnet Modal im Neuanlage-Modus (null = kein bestehendes Abo) -->
      <button class="btn btn-primary" id="btn-abo-neu" @click="openModal(null)">+ Neues Abo</button>
    </div>

    <!-- Statistik-Kacheln (computed-Werte) -->
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon blue">📋</div>
        <!-- JavaScript: abos.length aus dem props-Array, reaktiv aktualisiert -->
        <div class="stat-value">{{ abos.length }}</div>
        <div class="stat-label">Abotypen gesamt</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon green">💰</div>
        <div class="stat-value">{{ minPreis }} €</div>
        <div class="stat-label">Günstigstes Abo</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon teal">💎</div>
        <div class="stat-value">{{ maxPreis }} €</div>
        <div class="stat-label">Teuerstes Abo</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon orange">📊</div>
        <div class="stat-value">{{ avgPreis }} €</div>
        <div class="stat-label">Ø Grundpreis</div>
      </div>
    </div>

    <!-- Tabellen-Ansicht -->
    <div class="card">
      <div class="card-header">
        <div>
          <div class="card-title">Alle Abotypen</div>
          <div class="card-subtitle">Übersicht aller angelegten Abonnements</div>
        </div>
      </div>
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>AboNr</th><th>Grundpreis</th><th>Kündigung</th>
              <th>Laufzeit</th><th>Kurs</th><th>Getränke</th><th>Aktionen</th>
            </tr>
          </thead>
          <tbody>
            <!-- JavaScript: v-for mit Schlüssel-Präfix 'tbl-' um DOM-Konflikte zu vermeiden -->
            <tr v-for="abo in abos" :key="'tbl-' + abo.abonr">
              <td class="font-bold">#{{ abo.abonr }}</td>
              <td style="color:var(--clr-accent);font-weight:700;">{{ formatPreis(abo.grundpreis) }} €</td>
              <td class="td-muted">
                {{ abo.kuendigsfrist ? new Date(abo.kuendigsfrist).toLocaleDateString('de-DE') : '—' }}
              </td>
              <td class="td-muted">{{ laufzeitText(abo.laufzeit) }}</td>
              <!-- JavaScript: :class Binding wählt Badge-Farbe dynamisch -->
              <td><span :class="abo.kurs ? 'badge badge-success' : 'badge badge-neutral'">{{ abo.kurs ? 'Ja' : 'Nein' }}</span></td>
              <td><span :class="abo.getraenke ? 'badge badge-success' : 'badge badge-neutral'">{{ abo.getraenke ? 'Ja' : 'Nein' }}</span></td>
              <td>
                <div class="flex gap-sm">
                  <button class="btn btn-sm btn-ghost" @click="openModal(abo)">✏️ Bearbeiten</button>
                  <button class="btn btn-sm btn-danger" @click="loescheAbo(abo.abonr)">🗑</button>
                </div>
              </td>
            </tr>

            <!-- Leer-Zustand -->
            <tr v-if="abos.length === 0">
              <td colspan="7">
                <div class="empty-state">
                  <div class="empty-state-icon">📋</div>
                  <h3>Keine Abotypen vorhanden</h3>
                  <p>Lege das erste Abo an, um loszulegen.</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- MODAL: Abo anlegen / bearbeiten -->
    <!-- JavaScript: v-if zeigt Modal nur wenn showModal true ist -->
    <div class="modal-overlay" v-if="showModal" @click.self="closeModal" id="modal-abo-overlay">
      <div class="modal" id="modal-abo">
        <div class="modal-header">
          <h3 class="modal-title">{{ editAbo ? 'Abo bearbeiten' : 'Neues Abo anlegen' }}</h3>
          <button class="btn btn-icon btn-ghost" id="btn-abo-modal-close" @click="closeModal">✕</button>
        </div>

        <!-- JavaScript: @submit.prevent verhindert Standard-Formular-Reload -->
        <form @submit.prevent="speichereAbo" id="form-abo">
          <div class="form-grid">

            <div class="form-group">
              <label for="abo-input-nr">AboNr (automatisch)</label>
              <input id="abo-input-nr" type="text" v-model="form.abonr"
                     placeholder="Wird automatisch generiert" readonly disabled />
            </div>

            <div class="form-group">
              <label for="abo-input-preis">Grundpreis (€/Monat) *</label>
              <!-- JavaScript: v-model.number konvertiert Eingabe automatisch zu Number -->
              <input id="abo-input-preis" type="number" v-model.number="form.grundpreis"
                     required min="0" step="0.01" placeholder="0.00" />
            </div>

            <div class="form-group">
              <label for="abo-input-kuendigung">Kündigungsfrist (Datum) *</label>
              <!-- JavaScript: v-model bindet das Datum-Eingabefeld (Format YYYY-MM-DD) -->
              <input id="abo-input-kuendigung" type="date" v-model="form.kuendigsfristDate" required />
            </div>

            <div class="form-group">
              <label for="abo-input-laufzeit">Laufzeit *</label>
              <select id="abo-input-laufzeit" v-model="form.laufzeit" @change="berechneKuendigung">
                <option value="0-1">1 Monat</option>
                <option value="0-3">3 Monate</option>
                <option value="0-6">6 Monate</option>
                <option value="1-0">1 Jahr</option>
                <option value="2-0">2 Jahre</option>
              </select>
            </div>

            <div class="form-group full">
              <label>Inkludierte Leistungen</label>
              <div class="flex gap-md" style="flex-wrap:wrap;margin-top:4px;">
                <!-- JavaScript: v-model.number wandelt Checkbox-Wert zu 0/1 (für Oracle) -->
                <!-- true-value und false-value setzen den Wert bei checked/unchecked -->
                <label class="checkbox-group">
                  <input type="checkbox" id="abo-check-kurs" v-model="form.kurs" />
                  <span>🏋️ Kurs inklusive</span>
                </label>
                <label class="checkbox-group">
                  <input type="checkbox" id="abo-check-getraenke" v-model="form.getraenke" />
                  <span>🥤 Getränke inklusive</span>
                </label>
              </div>
            </div>

          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-ghost" id="btn-abo-abbrechen" @click="closeModal">Abbrechen</button>
            <button type="submit" class="btn btn-primary" id="btn-abo-speichern">
              {{ editAbo ? 'Speichern' : 'Anlegen' }}
            </button>
          </div>
        </form>
      </div>
    </div>

  </div>
</template>

<script>
/* =========================================================
   ██ JAVASCRIPT – AboVerwaltung.vue
   CRUD für Abos über die C# API.
   WICHTIG – Oracle-Datentypen:
     kurs, getraenke: NUMBER(1) → int 0 oder 1 (nicht bool!)
     kuendigsfrist:   DATE → ISO-String in der API
     laufzeit:        INTERVAL YEAR TO MONTH → String "Jahr-Monat" (z.B. "1-0")
   ========================================================= */

export default {
  name: 'AboVerwaltung',

  props: {
    abos: { type: Array, default: () => [] }
  },

  emits: ['abos-updated', 'show-toast'],

  data() {
    return {
      showModal: false,
      editAbo:   null,

      // Formularfelder – camelCase wie die C# API
      form: {
        abonr:             '',
        grundpreis:        0,
        kuendigsfristDate: '', // lokaler Wert im Format YYYY-MM-DD (für type="date")
        kurs:              false,  // C# erwartet Boolean
        getraenke:         false,  // C# erwartet Boolean
        laufzeit:          '1-0'
      }
    }
  },

  computed: {
    // Preisstatistiken für die Kacheln
    minPreis() {
      if (!this.abos.length) return '0,00'
      return Math.min(...this.abos.map(a => a.grundpreis || 0)).toFixed(2).replace('.', ',')
    },
    maxPreis() {
      if (!this.abos.length) return '0,00'
      return Math.max(...this.abos.map(a => a.grundpreis || 0)).toFixed(2).replace('.', ',')
    },
    avgPreis() {
      if (!this.abos.length) return '0,00'
      const sum = this.abos.reduce((s, a) => s + (parseFloat(a.grundpreis) || 0), 0)
      return (sum / this.abos.length).toFixed(2).replace('.', ',')
    }
  },

  methods: {
    // Öffnet das Modal zum Bearbeiten oder Neuanlegen
    openModal(abo) {
      this.editAbo = abo
      if (abo) {
        // ISO-Datum für type="date" kürzen: "2026-12-31T00:00:00Z" → "2026-12-31"
        let datumStr = ''
        if (abo.kuendigsfrist) {
          const parsed = new Date(abo.kuendigsfrist)
          if (!isNaN(parsed.getTime())) {
            datumStr = parsed.toISOString().split('T')[0]
          }
        }
        this.form = {
          abonr:             abo.abonr,
          grundpreis:        abo.grundpreis,
          kuendigsfristDate: datumStr,
          kurs:              !!abo.kurs,
          getraenke:         !!abo.getraenke,
          laufzeit:          abo.laufzeit  || '1-0'
        }
      } else {
        this.form = { abonr: '', grundpreis: 0, kuendigsfristDate: '', kurs: false, getraenke: false, laufzeit: '1-0' }
        this.berechneKuendigung()
      }
      this.showModal = true
    },

    closeModal() { this.showModal = false; this.editAbo = null },

    berechneKuendigung() {
      const heute = new Date()
      let jahre = 1
      let monate = 0
      
      if (this.form.laufzeit) {
        const parts = this.form.laufzeit.split('-')
        if (parts.length === 2) {
          jahre = parseInt(parts[0]) || 0
          monate = parseInt(parts[1]) || 0
        }
      }
      
      const zieldatum = new Date(heute.getFullYear() + jahre, heute.getMonth() + monate, heute.getDate())
      
      const y = zieldatum.getFullYear()
      const m = String(zieldatum.getMonth() + 1).padStart(2, '0')
      const d = String(zieldatum.getDate()).padStart(2, '0')
      this.form.kuendigsfristDate = `${y}-${m}-${d}`
    },

    /* ── speichereAbo() ─────────────────────────────────────────────
       Sendet POST (Neuanlage) oder PUT (Bearbeiten).
       Konvertiert das Datum zurück in ein ISO-Format für die API.
    ──────────────────────────────────────────────────────────────── */
    async speichereAbo() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'

      // Datum von YYYY-MM-DD (HTML input) prüfen und direkt verwenden.
      // Die API akzeptiert das "YYYY-MM-DD"-Format direkt und wandelt es in ein DateTime-Objekt um.
      let kuendigsfristFormatted = null
      if (this.form.kuendigsfristDate) {
        kuendigsfristFormatted = this.form.kuendigsfristDate
      } else {
        this.$emit('show-toast', { text: 'Bitte ein Kündigungsdatum auswählen.', type: 'danger' })
        return
      }

      // Body für die API (camelCase-Felder, kurs/getraenke als int)
      const body = {
        abonr:        this.form.abonr || 0,
        grundpreis:   this.form.grundpreis,
        kuendigsfrist: kuendigsfristFormatted,
        kurs:         this.form.kurs,
        getraenke:    this.form.getraenke,
        laufzeit:     this.form.laufzeit
      }

      const url    = this.editAbo ? `${baseUrl}/abos/${this.form.abonr}` : `${baseUrl}/abos`
      const method = this.editAbo ? 'PUT' : 'POST'

      try {
        const res = await fetch(url, {
          method,
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(body)
        })

        if (res.ok) {
          const resAbos = await fetch(`${baseUrl}/abos`)
          if (resAbos.ok) this.$emit('abos-updated', await resAbos.json())
          this.$emit('show-toast', { text: `Abo erfolgreich gespeichert.`, type: 'success' })
        } else {
          const fehler = await res.text()
          this.$emit('show-toast', { text: `Fehler: ${fehler}`, type: 'danger' })
        }
      } catch (err) {
        console.warn('API offline – lokale Änderung.', err)
        let neueAbos = [...this.abos]
        if (this.editAbo) {
          const idx = neueAbos.findIndex(a => a.abonr === this.form.abonr)
          if (idx !== -1) neueAbos[idx] = { ...neueAbos[idx], ...body }
        } else {
          neueAbos.push({ ...body, abonr: 'TEMP-' + Date.now() })
        }
        this.$emit('abos-updated', neueAbos)
        this.$emit('show-toast', { text: 'API offline. Lokal gespeichert.', type: 'warning' })
      }

      this.closeModal()
    },

    // Löscht ein Abo per DELETE-Request
    async loescheAbo(nr) {
      if (!confirm(`Abo #${nr} wirklich löschen?`)) return

      const neueAbos = this.abos.filter(a => a.abonr !== nr)
      this.$emit('abos-updated', neueAbos)

      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        const res = await fetch(`${baseUrl}/abos/${nr}`, { method: 'DELETE' })
        if (res.ok) {
          this.$emit('show-toast', { text: `Abo #${nr} gelöscht.`, type: 'success' })
        } else {
          this.$emit('show-toast', { text: `API-Fehler: ${res.status}`, type: 'danger' })
        }
      } catch (err) {
        this.$emit('show-toast', { text: `Abo #${nr} lokal gelöscht (API offline).`, type: 'warning' })
      }
    },

    // Konvertiert das Oracle INTERVAL-Format "Jahr-Monat" in lesbaren Text
    laufzeitText(laufzeit) {
      if (!laufzeit) return '—'
      const [jahre, monate] = laufzeit.split('-').map(Number)
      const teile = []
      if (jahre  > 0) teile.push(`${jahre} Jahr${jahre  > 1 ? 'e' : ''}`)
      if (monate > 0) teile.push(`${monate} Monat${monate > 1 ? 'e' : ''}`)
      return teile.join(' ') || '—'
    },

    // Formatiert Preis als deutsches Format
    formatPreis(p) {
      return parseFloat(p || 0).toFixed(2).replace('.', ',')
    }
  }
}
</script>

<style scoped>
.ml-auto { margin-left:auto; }
</style>
