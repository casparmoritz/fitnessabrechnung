<template>
  <!-- ==========================================
    AboVerwaltung.vue
    MUSS-KRITERIUM: Aboverträge anlegen, bearbeiten, löschen
    Optionen: Kurse, Getränke, Sauna (Ja/Nein)
    ========================================== -->
  <div class="page">

    <div class="page-header">
      <div>
        <h2 class="page-title">Abo-Verwaltung</h2>
        <p class="page-desc">Abotypen mit Grundpreisen und Zusatzoptionen verwalten</p>
      </div>
      <button class="btn btn-primary" id="btn-abo-neu" @click="openModal(null)">
        + Neues Abo
      </button>
    </div>

    <!-- Stat-Kacheln -->
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon blue">📋</div>
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

    <!-- Abo-Karten Grid -->
    <div class="abo-cards-grid" id="abo-grid">
      <div
        class="abo-card"
        v-for="abo in abos"
        :key="abo.aboNr"
        :id="'card-abo-' + abo.aboNr"
      >
        <!-- Header -->
        <div class="abo-card-header">
          <div>
            <div class="abo-card-nr">Abo</div>
            <div class="abo-card-name"># {{ abo.aboNr }}</div>
          </div>
          <div class="flex gap-sm">
            <button class="btn btn-sm btn-ghost" :id="'btn-edit-abo-' + abo.aboNr" @click="openModal(abo)">✏️</button>
            <button class="btn btn-sm btn-danger" :id="'btn-del-abo-' + abo.aboNr" @click="loescheAbo(abo.aboNr)">🗑</button>
          </div>
        </div>

        <!-- Preis -->
        <div class="abo-card-price">
          {{ formatPreis(abo.grundpreis) }} <span>€/Monat</span>
        </div>

        <!-- Laufzeit -->
        <div class="text-muted" style="font-size:12px; margin-bottom:12px;">
          Kündigungsfrist: {{ abo.kuendigungsfrist || '—' }}
        </div>

        <!-- Optionen -->
        <div class="abo-options">
          <div class="abo-option" :class="{ active: abo.kurs }">
            <span>🏋️</span> Kurs
            <span class="ml-auto">{{ abo.kurs ? '✔' : '✗' }}</span>
          </div>
          <div class="abo-option" :class="{ active: abo.getraenke }">
            <span>🥤</span> Getränke
            <span class="ml-auto">{{ abo.getraenke ? '✔' : '✗' }}</span>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-if="abos.length === 0" class="card" style="grid-column: 1/-1;">
        <div class="empty-state">
          <div class="empty-state-icon">📋</div>
          <h3>Keine Abotypen vorhanden</h3>
          <p>Lege das erste Abo an, um loszulegen.</p>
        </div>
      </div>
    </div>

    <!-- Detaillierte Tabellen-Ansicht -->
    <div class="card mt-lg">
      <div class="card-header">
        <div>
          <div class="card-title">Alle Abos – Tabellenansicht</div>
          <div class="card-subtitle">Detaillierter Überblick aller Abotypen</div>
        </div>
      </div>
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>AboNr</th>
              <th>Grundpreis</th>
              <th>Kündigung</th>
              <th>Kurs</th>
              <th>Getränke</th>
              <th>Aktionen</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="abo in abos" :key="'tbl-' + abo.aboNr">
              <td class="font-bold">#{{ abo.aboNr }}</td>
              <td style="color:var(--clr-accent); font-weight:700;">{{ formatPreis(abo.grundpreis) }} €</td>
              <td class="td-muted">{{ abo.kuendigungsfrist || '—' }}</td>
              <td><span :class="abo.kurs ? 'badge badge-success' : 'badge badge-neutral'">{{ abo.kurs ? 'Ja' : 'Nein' }}</span></td>
              <td><span :class="abo.getraenke ? 'badge badge-success' : 'badge badge-neutral'">{{ abo.getraenke ? 'Ja' : 'Nein' }}</span></td>
              <td>
                <div class="flex gap-sm">
                  <button class="btn btn-sm btn-ghost" @click="openModal(abo)">✏️ Bearbeiten</button>
                  <button class="btn btn-sm btn-danger" @click="loescheAbo(abo.aboNr)">🗑</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ==========================================
      MODAL: Abo anlegen / bearbeiten
    ========================================== -->
    <div class="modal-overlay" v-if="showModal" @click.self="closeModal" id="modal-abo-overlay">
      <div class="modal" id="modal-abo">
        <div class="modal-header">
          <h3 class="modal-title">{{ editAbo ? 'Abo bearbeiten' : 'Neues Abo anlegen' }}</h3>
          <button class="btn btn-icon btn-ghost" id="btn-abo-modal-close" @click="closeModal">✕</button>
        </div>

        <form @submit.prevent="speichereAbo" id="form-abo">
          <div class="form-grid">

            <div class="form-group">
              <label for="abo-input-nr">AboNr</label>
              <input id="abo-input-nr" type="text" v-model="form.aboNr" :readonly="!!editAbo" placeholder="Automatisch" />
            </div>

            <div class="form-group">
              <label for="abo-input-preis">Grundpreis (€/Monat) *</label>
              <input id="abo-input-preis" type="number" v-model.number="form.grundpreis" required min="0" step="0.01" placeholder="0.00" />
            </div>

            <div class="form-group">
              <label for="abo-input-kuendigung">Kündigungsfrist (Datum) *</label>
              <input id="abo-input-kuendigung" type="date" v-model="form.kuendigungsfrist" required />
            </div>

            <!-- Optionen -->
            <div class="form-group full">
              <label>Inkludierte Leistungen</label>
              <div class="flex gap-md" style="flex-wrap:wrap; margin-top:4px;">
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
   JAVASCRIPT – AboVerwaltung.vue
   Zeigt Abo-Karten, Tabelle, Modal-Formular für CRUD
   Props:  abos[] (alle Abos aus App.vue)
   Emits:  'abos-updated' → aktualisiertes Array an App.vue
   ========================================================= */

export default {
  name: 'AboVerwaltung',

  // Props = Daten die von App.vue übergeben werden
  props: {
    abos: { type: Array, default: () => [] } // Liste aller Abos
  },

  // Welche Events diese Komponente nach oben senden darf
  emits: ['abos-updated'],

  // data() = alle reaktiven Variablen dieser Komponente
  data() {
    return {
      // Steuert ob das Modal-Formular sichtbar ist
      showModal: false,
      // null = Neuanlage, sonst das Abo-Objekt das bearbeitet wird
      editAbo: null,

      // Formular-Felder – mit v-model an die Eingabefelder im Modal gebunden
      form: {
        aboNr:            '',
        grundpreis:       0,
        kuendigungsfrist: '',
        kurs:             false,
        getraenke:        false
      }
    }
  },

  // computed = berechnete Eigenschaften für die Statistik-Kacheln
  computed: {
    // Günstigster Grundpreis aller Abos
    minPreis() {
      if (!this.abos.length) return '0,00'
      return Math.min(...this.abos.map(a => a.grundpreis || 0)).toFixed(2).replace('.', ',')
    },
    // Teuerster Grundpreis aller Abos
    maxPreis() {
      if (!this.abos.length) return '0,00'
      return Math.max(...this.abos.map(a => a.grundpreis || 0)).toFixed(2).replace('.', ',')
    },
    // Durchschnittlicher Grundpreis aller Abos
    avgPreis() {
      if (!this.abos.length) return '0,00'
      const sum = this.abos.reduce((s, a) => s + (parseFloat(a.grundpreis) || 0), 0)
      return (sum / this.abos.length).toFixed(2).replace('.', ',')
    }
  },

  methods: {
    // Öffnet das Modal-Formular:
    // - Bei null → Neuanlage (leeres Formular)
    // - Bei Abo-Objekt → Bearbeiten (Formular mit vorhandenen Daten befüllen)
    openModal(abo) {
      this.editAbo = abo
      if (abo) {
        this.form = { ...abo } // Kopie des Objekts
      } else {
        this.form = { aboNr: '', grundpreis: 0, kuendigungsfrist: '', kurs: false, getraenke: false }
      }
      this.showModal = true
    },

    // Schließt das Modal und setzt editAbo zurück
    closeModal() { this.showModal = false; this.editAbo = null },

    // ✅ IMPLEMENTIERT: Abo speichern (Neuanlage = POST, Bearbeiten = PUT)
    async speichereAbo() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'

      // Bei Neuanlage: Oracle-DB vergibt aboNr automatisch
      // Bei Bearbeiten: aboNr als Pfad-Parameter (z.B. /abos/A001)
      const url = this.editAbo
        ? `${baseUrl}/abos/${this.form.aboNr}`
        : `${baseUrl}/abos`
      const method = this.editAbo ? 'PUT' : 'POST'

      try {
        // Formular-Daten als JSON an die API senden
        const res = await fetch(url, {
          method: method,
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.form)
        })

        if (res.ok) {
          // Nach Erfolg aktuelle Liste nachladen (enthält die echte AboNr der DB)
          const resAbos = await fetch(`${baseUrl}/abos`)
          if (resAbos.ok) {
            this.$emit('abos-updated', await resAbos.json())
          }
        }
      } catch (err) {
        // Offline-Fallback: Änderung nur lokal speichern
        console.warn('API offline – Änderung nur lokal.', err)
        let neueAbos = [...this.abos]
        if (this.editAbo) {
          const idx = neueAbos.findIndex(a => a.aboNr === this.form.aboNr)
          if (idx !== -1) neueAbos[idx] = { ...this.form }
        } else {
          // Temporäre Nummer damit der Eintrag im Browser sichtbar ist
          neueAbos.push({ ...this.form, aboNr: 'TEMP-' + Date.now() })
        }
        this.$emit('abos-updated', neueAbos)
      }

      this.closeModal()
    },

    // ✅ IMPLEMENTIERT: Abo löschen (lokal sofort, API im Hintergrund)
    async loescheAbo(nr) {
      if (!confirm(`Abo #${nr} wirklich löschen?`)) return

      // Sofort lokal entfernen → UI reagiert ohne Wartezeit
      const neueAbos = this.abos.filter(a => a.aboNr !== nr)
      this.$emit('abos-updated', neueAbos)

      // API im Hintergrund benachrichtigen
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        await fetch(`${baseUrl}/abos/${nr}`, { method: 'DELETE' })
      } catch (err) {
        console.warn('API offline. Löschen erfolgte nur lokal.')
      }
    },

    // Formatiert eine Zahl als deutsches Preis-Format (z.B. 49.90 → "49,90")
    formatPreis(p) {
      return parseFloat(p || 0).toFixed(2).replace('.', ',')
    }
  }
}
</script>

<style scoped>
/* Abo-Karten Grid */
.abo-cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: var(--sp-md);
  margin-bottom: var(--sp-lg);
}

.abo-card {
  background: var(--clr-surface);
  border: 1px solid var(--clr-border);
  border-radius: var(--rad-lg);
  padding: var(--sp-lg);
  transition: all 250ms ease;
}
.abo-card:hover {
  border-color: var(--clr-primary);
  box-shadow: 0 0 24px rgba(108,99,255,0.2);
  transform: translateY(-3px);
}

.abo-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: var(--sp-sm);
}
.abo-card-nr {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--clr-text-dim);
}
.abo-card-name {
  font-size: 17px;
  font-weight: 700;
  color: var(--clr-text);
  margin-top: 2px;
}
.abo-card-price {
  font-size: 28px;
  font-weight: 800;
  color: var(--clr-accent);
  letter-spacing: -0.03em;
  margin-bottom: 4px;
}
.abo-card-price span {
  font-size: 14px;
  color: var(--clr-text-muted);
  font-weight: 400;
}

.abo-options {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.abo-option {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 7px 10px;
  border-radius: var(--rad-sm);
  font-size: 13px;
  font-weight: 500;
  background: var(--clr-bg);
  color: var(--clr-text-muted);
  border: 1px solid var(--clr-border);
}
.abo-option.active {
  color: var(--clr-success);
  border-color: rgba(54,211,153,0.25);
  background: var(--clr-success-dim);
}
.ml-auto { margin-left: auto; }
</style>
