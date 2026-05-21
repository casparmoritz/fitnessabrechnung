<template>
  <!-- ==========================================
    KundenVerwaltung.vue
    MUSS-KRITERIUM: Kunden anlegen, bearbeiten, Abos zuweisen, IBAN-Zuordnung
    ========================================== -->
  <div class="page">

    <!-- Page Header -->
    <div class="page-header">
      <div>
        <h2 class="page-title">Kundenverwaltung</h2>
        <p class="page-desc">Kunden anlegen, bearbeiten und Aboverträge zuweisen</p>
      </div>
      <div class="flex gap-sm">
        <button class="btn btn-ghost" id="btn-kunden-export" @click="$emit('export')">
          ⬇ Export
        </button>
        <button class="btn btn-primary" id="btn-kunden-neu" @click="openModal(null)">
          + Neuer Kunde
        </button>
      </div>
    </div>

    <!-- Stat-Kacheln -->
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon blue">👤</div>
        <div class="stat-value">{{ kunden.length }}</div>
        <div class="stat-label">Kunden gesamt</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon green">✅</div>
        <div class="stat-value">{{ aktivKunden }}</div>
        <div class="stat-label">Aktive Abos</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon orange">🏷</div>
        <div class="stat-value">{{ ermaessigtKunden }}</div>
        <div class="stat-label">Ermäßigt</div>
      </div>
      <div class="stat-card">
        <div class="stat-icon red">⚠️</div>
        <div class="stat-value">{{ ohneAbo }}</div>
        <div class="stat-label">Ohne Abo</div>
      </div>
    </div>

    <!-- Filter-Bar -->
    <div class="filter-bar">
      <div class="search-input-wrap">
        <span class="search-icon">🔍</span>
        <input
          id="kunden-suche"
          type="text"
          v-model="suchbegriff"
          placeholder="Name, KundenNr oder IBAN suchen …"
        />
      </div>
      <select id="filter-abo" v-model="filterAbo" style="width:auto; min-width:160px;">
        <option value="">Alle Abos</option>
        <option v-for="abo in abos" :key="abo.aboNr" :value="abo.aboNr">
          Abo #{{ abo.aboNr }}
        </option>
      </select>
      <select id="filter-status" v-model="filterStatus" style="width:auto; min-width:140px;">
        <option value="">Alle Status</option>
        <option value="aktiv">Aktiv</option>
        <option value="ermaessigt">Ermäßigt</option>
        <option value="ohneAbo">Ohne Abo</option>
      </select>
    </div>

    <!-- Kundentabelle -->
    <div class="card">
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>KundenNr</th>
              <th>Vorname</th>
              <th>Nachname</th>
              <th>IBAN</th>
              <th>BIC</th>
              <th>Bank</th>
              <th>AboNr</th>
              <th>Ermäßigung</th>
              <th>Monatsbeitrag</th>
              <th>Aktionen</th>
            </tr>
          </thead>
          <tbody>
            <!-- Zeile pro Kunde – gefiltertes Array per computed "gefilterteKunden" -->
            <tr v-for="kunde in gefilterteKunden" :key="kunde.kundenNr" :id="'row-kd-' + kunde.kundenNr">
              <td class="td-mono">{{ kunde.kundenNr }}</td>
              <td class="font-bold">{{ kunde.vorname }}</td>
              <td class="font-bold">{{ kunde.nachname }}</td>
              <td class="td-mono">{{ ibanMask(kunde.iban) }}</td>
              <td class="td-mono">{{ kunde.bic || '—' }}</td>
              <td class="td-muted">{{ kunde.bank || '—' }}</td>
              <td>
                <span v-if="kunde.aboNr" class="badge badge-primary">
                  Abo #{{ kunde.aboNr }}
                </span>
                <span v-else class="badge badge-neutral">– kein Abo –</span>
              </td>
              <td>
                <span v-if="kunde.ermaessigungssatz" class="badge badge-warning">
                  {{ kunde.ermaessigungssatz }}%
                </span>
                <span v-else class="td-muted">—</span>
              </td>
              <td class="font-bold" style="color:var(--clr-accent)">
                {{ monatsbeitrag(kunde) }} €
              </td>
              <td>
                <div class="flex gap-sm">
                  <button class="btn btn-sm btn-ghost" :id="'btn-edit-kd-' + kunde.kundenNr" @click="openModal(kunde)">✏️</button>
                  <button class="btn btn-sm btn-danger" :id="'btn-del-kd-' + kunde.kundenNr" @click="loescheKunde(kunde.kundenNr)">🗑</button>
                </div>
              </td>
            </tr>

            <!-- Empty State -->
            <tr v-if="gefilterteKunden.length === 0">
              <td colspan="8">
                <div class="empty-state">
                  <div class="empty-state-icon">👤</div>
                  <h3>Keine Kunden gefunden</h3>
                  <p>Passe deinen Filter an oder lege einen neuen Kunden an.</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ==========================================
      MODAL: Kunden anlegen / bearbeiten
    ========================================== -->
    <div class="modal-overlay" v-if="showModal" @click.self="closeModal" id="modal-kunde-overlay">
      <div class="modal" id="modal-kunde">
        <div class="modal-header">
          <h3 class="modal-title">
            {{ editKunde ? 'Kunde bearbeiten' : 'Neuen Kunden anlegen' }}
          </h3>
          <button class="btn btn-icon btn-ghost" id="btn-modal-close" @click="closeModal">✕</button>
        </div>

        <!-- Formular -->
        <form @submit.prevent="speichereKunde" id="form-kunde">
          <div class="form-grid">

            <!-- KundenNr (automatisch / read-only beim Bearbeiten) -->
            <div class="form-group">
              <label for="input-kundennr">KundenNr</label>
              <input
                id="input-kundennr"
                type="text"
                v-model="form.kundenNr"
                placeholder="Automatisch"
                :readonly="!!editKunde"
              />
            </div>

            <!-- Vorname -->
            <div class="form-group">
              <label for="input-vorname">Vorname *</label>
              <input id="input-vorname" type="text" v-model="form.vorname" required placeholder="Max" />
            </div>

            <!-- Nachname -->
            <div class="form-group">
              <label for="input-nachname">Nachname *</label>
              <input id="input-nachname" type="text" v-model="form.nachname" required placeholder="Mustermann" />
            </div>

            <!-- IBAN -->
            <div class="form-group full">
              <label for="input-iban">IBAN (tbl_Konto) *</label>
              <input
                id="input-iban"
                type="text"
                v-model="form.iban"
                required
                minlength="22"
                placeholder="DE00 0000 0000 0000 0000 00"
                maxlength="22"
              />
            </div>

            <!-- BIC & Bank -->
            <div class="form-group">
              <label for="input-bic">BIC (tbl_Bank) *</label>
              <input id="input-bic" type="text" v-model="form.bic" required placeholder="z.B. GENODEF1XXX" />
            </div>
            <div class="form-group">
              <label for="input-bank">Bank Name *</label>
              <input id="input-bank" type="text" v-model="form.bank" required placeholder="z.B. Musterbank" />
            </div>

            <!-- Abo zuweisen -->
            <div class="form-group">
              <label for="select-abo">Abo zuweisen</label>
              <select id="select-abo" v-model="form.aboNr">
                <option value="">– Kein Abo –</option>
                <option v-for="abo in abos" :key="abo.aboNr" :value="abo.aboNr">
                  Abo #{{ abo.aboNr }} ({{ abo.grundpreis }} €/Monat)
                </option>
              </select>
            </div>

            <!-- Ermäßigung (Dropdown aus tbl_Ermaessigung via API) -->
            <div class="form-group">
              <label for="select-ermaessigung">Ermäßigung</label>
              <select
                id="select-ermaessigung"
                v-model.number="form.ermaessigungssatz"
              >
                <!-- Standardoption: keine Ermäßigung -->
                <option :value="0">– Keine Ermäßigung –</option>
                <!-- Eine Option pro Ermäßigungsart aus der API -->
                <option
                  v-for="erm in ermaessigungen"
                  :key="erm.ermID"
                  :value="erm.ermaessigungssatz"
                >
                  {{ erm.bezeichnung }} ({{ erm.ermaessigungssatz }}%)
                </option>
              </select>
            </div>

          </div>

          <!-- Berechneter Preis Vorschau -->
          <div class="alert alert-info mt-md" v-if="form.aboNr">
            💡 Monatlicher Beitrag:
            <strong>{{ preisVorschau }} €</strong>
            <span v-if="form.ermaessigungssatz > 0">
              (inkl. {{ form.ermaessigungssatz }}% Ermäßigung)
            </span>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-ghost" id="btn-modal-abbrechen" @click="closeModal">Abbrechen</button>
            <button type="submit" class="btn btn-primary" id="btn-modal-speichern">
              {{ editKunde ? 'Speichern' : 'Anlegen' }}
            </button>
          </div>
        </form>
      </div>
    </div>

  </div>
</template>

<script>
/* =========================================================
   JAVASCRIPT – KundenVerwaltung.vue
   Zeigt Kundenliste, Filter, Modal-Formular für CRUD
   Props:  kunden[] (alle Kunden), abos[] (für Abo-Dropdown)
   Emits:  'kunden-updated' → aktualisiertes Array an App.vue
   ========================================================= */

export default {
  name: 'KundenVerwaltung',

  // Props = Daten die von der übergeordneten Komponente (App.vue) übergeben werden
  props: {
    kunden:         { type: Array, default: () => [] }, // Liste aller Kunden
    abos:           { type: Array, default: () => [] }, // Liste aller Abos (für das Dropdown im Modal)
    ermaessigungen: { type: Array, default: () => [] }  // ✅ IMPLEMENTIERT: Ermäßigungen aus tbl_Ermaessigung (für Dropdown)
  },

  // Welche Events diese Komponente nach oben senden darf
  emits: ['kunden-updated', 'export'],

  // data() = alle reaktiven Variablen dieser Komponente
  data() {
    return {
      // Suchwort aus dem Suchfeld oben
      suchbegriff:  '',
      // Aktuell gewählter Abo-Filter (leer = alle)
      filterAbo:    '',
      // Aktuell gewählter Status-Filter (leer = alle)
      filterStatus: '',

      // Steuert ob das Modal (Formular-Popup) sichtbar ist
      showModal: false,
      // null = Neuanlage, sonst das Kunden-Objekt das bearbeitet wird
      editKunde: null,

      // Formular-Felder – werden mit v-model an die Input-Felder gebunden
      form: {
        kundenNr:          '',
        vorname:           '',
        nachname:          '',
        iban:              '',
        bic:               '',
        bank:              '',
        aboNr:             '',
        ermaessigungssatz: 0
      }
    }
  },

  // computed = berechnete Eigenschaften, werden bei Änderung automatisch aktualisiert
  computed: {
    // Gibt die Kundenliste gefiltert nach Suchwort, Abo und Status zurück
    gefilterteKunden() {
      return this.kunden.filter(k => {
        const q = this.suchbegriff.toLowerCase()
        // Prüft ob der Suchbegriff in einem der Felder vorkommt
        const matchSearch = !q ||
          k.vorname?.toLowerCase().includes(q) ||
          k.nachname?.toLowerCase().includes(q) ||
          String(k.kundenNr).includes(q) ||
          k.iban?.toLowerCase().replace(/\s/g,'').includes(q.replace(/\s/g,'')) ||
          k.bic?.toLowerCase().includes(q) ||
          k.bank?.toLowerCase().includes(q)

        // Prüft ob der Abo-Filter zutrifft
        const matchAbo = !this.filterAbo || k.aboNr === this.filterAbo

        // Prüft ob der Status-Filter zutrifft
        let matchStatus = true
        if (this.filterStatus === 'aktiv')      matchStatus = !!k.aboNr
        if (this.filterStatus === 'ermaessigt') matchStatus = !!k.ermaessigungssatz
        if (this.filterStatus === 'ohneAbo')    matchStatus = !k.aboNr

        return matchSearch && matchAbo && matchStatus
      })
    },

    // Anzahl Kunden mit einem aktiven Abo (für die Statistik-Kachel)
    aktivKunden()      { return this.kunden.filter(k =>  k.aboNr).length },
    // Anzahl Kunden mit einer Ermäßigung > 0
    ermaessigtKunden() { return this.kunden.filter(k =>  k.ermaessigungssatz > 0).length },
    // Anzahl Kunden ohne Abo
    ohneAbo()          { return this.kunden.filter(k => !k.aboNr).length },

    // Berechnet den Monatsbeitrag live im Modal (Grundpreis - Ermäßigung %)
    preisVorschau() {
      const abo = this.abos.find(a => a.aboNr === this.form.aboNr)
      if (!abo) return '0,00'
      const basis = parseFloat(abo.grundpreis) || 0
      const rabatt = parseFloat(this.form.ermaessigungssatz) || 0
      return (basis * (1 - rabatt / 100)).toFixed(2).replace('.', ',')
    }
  },

  methods: {
    // Öffnet das Modal-Formular:
    // - Bei null → Neuanlage (leeres Formular)
    // - Bei Kunden-Objekt → Bearbeiten (Formular mit vorhandenen Daten befüllen)
    openModal(kunde) {
      this.editKunde = kunde
      if (kunde) {
        this.form = { ...kunde } // Kopie des Objekts, damit Änderungen nicht sofort übernommen werden
      } else {
        this.form = { kundenNr: '', vorname: '', nachname: '', iban: '', bic: '', bank: '', aboNr: '', ermaessigungssatz: 0 }
      }
      this.showModal = true
    },

    // Schließt das Modal und setzt editKunde zurück
    closeModal() {
      this.showModal = false
      this.editKunde = null
    },

    // ✅ IMPLEMENTIERT: Kunden speichern (Neuanlage = POST, Bearbeiten = PUT)
    async speichereKunde() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'

      // Bei Neuanlage: Oracle-DB vergibt kundenNr automatisch → kein Pfad-Parameter
      // Bei Bearbeiten: kundenNr als Pfad-Parameter übergeben (z.B. /kunden/K001)
      const url = this.editKunde
        ? `${baseUrl}/kunden/${this.form.kundenNr}`
        : `${baseUrl}/kunden`
      const method = this.editKunde ? 'PUT' : 'POST'

      try {
        // Formular-Daten als JSON an die API senden
        const res = await fetch(url, {
          method: method,
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.form)
        })

        if (res.ok) {
          // Nach Erfolg aktuelle Liste nachladen (enthält die echte KundenNr der DB)
          const resKunden = await fetch(`${baseUrl}/kunden`)
          if (resKunden.ok) {
            this.$emit('kunden-updated', await resKunden.json())
          }
          this.$emit('show-toast', {
            text: `Kunde ${this.form.vorname} ${this.form.nachname} erfolgreich gespeichert.`,
            type: 'success'
          })
        } else {
          this.$emit('show-toast', {
            text: `Fehler beim Speichern über die API. Status: ${res.status}`,
            type: 'danger'
          })
        }
      } catch (err) {
        // Offline-Fallback: Änderung nur lokal speichern
        console.warn('API offline – Änderung nur lokal.', err)
        let neueKunden = [...this.kunden]
        if (this.editKunde) {
          const idx = neueKunden.findIndex(k => k.kundenNr === this.form.kundenNr)
          if (idx !== -1) neueKunden[idx] = { ...this.form }
        } else {
          // Temporäre Nummer damit der Eintrag im Browser sichtbar ist
          neueKunden.push({ ...this.form, kundenNr: 'TEMP-' + Date.now() })
        }
        this.$emit('kunden-updated', neueKunden)
        this.$emit('show-toast', {
          text: 'API ist offline. Kunde wurde temporär lokal gespeichert.',
          type: 'warning'
        })
      }

      this.closeModal()
    },

    // ✅ IMPLEMENTIERT: Kunden löschen (lokal sofort, API im Hintergrund)
    async loescheKunde(nr) {
      if (!confirm(`Kunden #${nr} wirklich löschen?`)) return

      // Sofort lokal entfernen → UI reagiert ohne Wartezeit
      const neueKunden = this.kunden.filter(k => k.kundenNr !== nr)
      this.$emit('kunden-updated', neueKunden)

      // API im Hintergrund benachrichtigen
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        const res = await fetch(`${baseUrl}/kunden/${nr}`, { method: 'DELETE' })
        if (res.ok) {
          this.$emit('show-toast', {
            text: `Kunde #${nr} erfolgreich über die API gelöscht.`,
            type: 'success'
          })
        } else {
          this.$emit('show-toast', {
            text: `Kunde #${nr} lokal gelöscht, aber API-Fehler: ${res.status}`,
            type: 'danger'
          })
        }
      } catch (err) {
        console.warn('API offline. Löschen erfolgte nur lokal.')
        this.$emit('show-toast', {
          text: `Kunde #${nr} wurde nur lokal gelöscht (API offline).`,
          type: 'warning'
        })
      }
    },

    // Versteckt die IBAN – zeigt nur erste 4 und letzte 4 Zeichen (Datenschutz)
    ibanMask(iban) {
      if (!iban) return '—'
      const clean = iban.replace(/\s/g, '')
      return clean.slice(0, 4) + ' **** **** **** ' + clean.slice(-4)
    },

    // Berechnet den monatlichen Beitrag eines Kunden (Grundpreis minus Ermäßigung)
    monatsbeitrag(kunde) {
      const abo = this.abos.find(a => a.aboNr === kunde.aboNr)
      if (!abo) return '0,00'
      const basis = parseFloat(abo.grundpreis) || 0
      const rabatt = parseFloat(kunde.ermaessigungssatz) || 0
      return (basis * (1 - rabatt / 100)).toFixed(2).replace('.', ',')
    }
  }
}
</script>
