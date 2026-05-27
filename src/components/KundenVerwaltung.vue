<template>
  <!-- ==========================================
    KundenVerwaltung.vue
    MUSS-KRITERIUM: Kunden anlegen, bearbeiten, Abos zuweisen, IBAN-Zuordnung
    Feldnamen entsprechen der C# API (camelCase): kundennr, vorname, nachname, iban...
    ========================================== -->
  <div class="page">

    <div class="page-header">
      <div>
        <h2 class="page-title">Kundenverwaltung</h2>
        <p class="page-desc">Kunden anlegen, bearbeiten und Aboverträge zuweisen</p>
      </div>
      <div class="flex gap-sm">
        <!-- JavaScript: @click-Binding ruft exportKunden() auf -->
        <button class="btn btn-ghost" id="btn-kunden-export" @click="exportKunden">⬇ Export</button>
        <!-- JavaScript: @click mit null = Neuanlage-Modus -->
        <button class="btn btn-primary" id="btn-kunden-neu" @click="openModal(null)">+ Neuer Kunde</button>
      </div>
    </div>

    <!-- Statistik-Kacheln (computed-Werte) -->
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon blue">👤</div>
        <!-- JavaScript: computed 'kunden.length' reaktiv -->
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
        <!-- JavaScript: v-model bindet suchbegriff reaktiv ans Eingabefeld -->
        <input id="kunden-suche" type="text" v-model="suchbegriff"
               placeholder="Name, KundenNr oder IBAN suchen …" />
      </div>
      <!-- JavaScript: v-model bindet filterAbo; v-for erzeugt Optionen aus abos-Array -->
      <select id="filter-abo" v-model="filterAboNr" style="width:auto;min-width:160px;">
        <option value="">Alle Abos</option>
        <option v-for="abo in abos" :key="abo.abonr" :value="abo.abonr">
          Abo #{{ abo.abonr }} ({{ formatPreis(abo.grundpreis) }} €)
        </option>
      </select>
    </div>

    <!-- Kundentabelle -->
    <div class="card">
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th @click="sortiereNach('kundennr')" style="cursor:pointer;user-select:none;">
                KundenNr <span v-if="sortBy === 'kundennr'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th @click="sortiereNach('vorname')" style="cursor:pointer;user-select:none;">
                Vorname <span v-if="sortBy === 'vorname'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th @click="sortiereNach('nachname')" style="cursor:pointer;user-select:none;">
                Nachname <span v-if="sortBy === 'nachname'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th @click="sortiereNach('iban')" style="cursor:pointer;user-select:none;">
                IBAN <span v-if="sortBy === 'iban'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th @click="sortiereNach('bic')" style="cursor:pointer;user-select:none;">
                BIC <span v-if="sortBy === 'bic'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th @click="sortiereNach('bank')" style="cursor:pointer;user-select:none;">
                Bank <span v-if="sortBy === 'bank'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th @click="sortiereNach('ermaessigung')" style="cursor:pointer;user-select:none;">
                Ermäßigung <span v-if="sortBy === 'ermaessigung'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th>Aktionen</th>
            </tr>
          </thead>
          <tbody>
            <!-- JavaScript: v-for über computed 'gefilterteKunden' -->
            <tr v-for="kunde in gefilterteKunden" :key="kunde.kundennr" :id="'row-kd-' + kunde.kundennr">
              <td class="td-mono">{{ kunde.kundennr }}</td>
              <td class="font-bold">{{ kunde.vorname }}</td>
              <td class="font-bold">{{ kunde.nachname }}</td>
              <!-- IBAN aus dem Konto-Navigationsobjekt (kunde.konto.iban oder direkt kunde.iban) -->
              <td class="td-mono">{{ ibanMask(kunde.iban || kunde.konto?.iban) }}</td>
              <!-- BIC und Bankname kommen über das verschachtelte Navigationsobjekt -->
              <td class="td-mono">{{ kunde.konto?.bic || '—' }}</td>
              <td class="td-muted">{{ kunde.konto?.bank?.bankName || '—' }}</td>
              <td>
                <span v-if="kunde.ermaessigung && kunde.ermaessigung.ermaessigungssatz > 0" class="badge badge-warning">
                  {{ getErmaessigungName(kunde.ermaessigung.ermid) }} ({{ (kunde.ermaessigung.ermaessigungssatz * 100).toFixed(0) }}%)
                </span>
                <span v-else class="td-muted">—</span>
              </td>
              <td>
                <div class="flex gap-sm">
                  <!-- JavaScript: @click übergibt das aktuelle kunde-Objekt ans Modal -->
                  <button class="btn btn-sm btn-ghost" :id="'btn-edit-kd-' + kunde.kundennr"
                          @click="openModal(kunde)">✏️</button>
                  <button class="btn btn-sm btn-danger" :id="'btn-del-kd-' + kunde.kundennr"
                          @click="loescheKunde(kunde.kundennr)">🗑</button>
                </div>
              </td>
            </tr>

            <!-- Leer-Zustand -->
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
    <!-- JavaScript: v-if steuert Sichtbarkeit; @click.self schließt bei Klick außerhalb -->
    <div class="modal-overlay" v-if="showModal" @click.self="closeModal" id="modal-kunde-overlay">
      <div class="modal" id="modal-kunde">
        <div class="modal-header">
          <h3 class="modal-title">
            {{ editKunde ? 'Kunde bearbeiten' : 'Neuen Kunden anlegen' }}
          </h3>
          <button class="btn btn-icon btn-ghost" id="btn-modal-close" @click="closeModal">✕</button>
        </div>

        <!-- JavaScript: @submit.prevent verhindert normales HTML-Formular-Submit -->
        <form @submit.prevent="speichereKunde" id="form-kunde">
          <div class="form-grid">

            <div class="form-group">
              <label for="input-kundennr">KundenNr (automatisch)</label>
              <input id="input-kundennr" type="text" v-model="form.kundennr"
                     placeholder="Wird automatisch generiert" readonly disabled />
            </div>

            <div class="form-group">
              <label for="input-vorname">Vorname *</label>
              <!-- JavaScript: v-model bindet form.vorname bidirektional -->
              <input id="input-vorname" type="text" v-model="form.vorname" required placeholder="Max" />
            </div>

            <div class="form-group">
              <label for="input-nachname">Nachname *</label>
              <input id="input-nachname" type="text" v-model="form.nachname" required placeholder="Mustermann" />
            </div>

            <!-- IBAN + BIC: aus tbl_Konto -->
            <div class="form-group full">
              <label for="input-iban">IBAN (aus tbl_Konto) *</label>
              <input id="input-iban" type="text" v-model="form.iban" required
                     @input="form.iban = form.iban.replace(/\s/g, '').toUpperCase()"
                     placeholder="DE00000000000000000000" minlength="22" maxlength="22" />
            </div>

            <div class="form-group">
              <label for="input-bic">BIC (aus tbl_Bank) *</label>
              <!-- JavaScript: Autovervollständigung aus Konten-Array beim Tippen -->
              <input id="input-bic" type="text" v-model="form.bic" required
                     @input="onBicInput"
                     placeholder="z.B. SPKADEFFFXX" list="bic-list" />
              <!-- JavaScript: datalist generiert Vorschläge aus dem banken-Array -->
              <datalist id="bic-list">
                <option v-for="bank in banken" :key="bank.bic" :value="bank.bic">{{ bank.bankName }}</option>
              </datalist>
            </div>

            <div class="form-group">
              <label for="input-bank">Bankname *</label>
              <input id="input-bank" type="text" v-model="form.bankName" required placeholder="z.B. Sparkasse" />
            </div>

            <div class="form-group">
              <label for="input-abo">Abo-Vertrag (optional)</label>
              <select id="input-abo" v-model.number="form.abonr">
                <option :value="0">Kein Abo</option>
                <option v-for="abo in abos" :key="abo.abonr" :value="abo.abonr">
                  Abo #{{ abo.abonr }} ({{ formatPreis(abo.grundpreis) }} €)
                </option>
              </select>
            </div>

            <div class="form-group">
              <label for="input-ermaessigung">Ermäßigung (optional)</label>
              <select id="input-ermaessigung" v-model.number="form.ermid">
                <option :value="0">Keine Ermäßigung</option>
                <option v-for="erm in ermaessigungen" :key="erm.ermid" :value="erm.ermid">
                  {{ getErmaessigungName(erm.ermid) }} ({{ (erm.ermaessigungssatz * 100).toFixed(0) }}%)
                </option>
              </select>
            </div>

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
   ██ JAVASCRIPT – KundenVerwaltung.vue
   CRUD für Kunden über die C# API.
   WICHTIG: C# API liefert camelCase-Felder:
     kundennr  (nicht KundenNr)
     vorname   (nicht Vorname)
     nachname  (nicht Nachname)
     iban      (nicht IBAN)
     konto     (Navigation: { bic, bank: { bankName } })
   ========================================================= */

export default {
  name: 'KundenVerwaltung',

  // Props: Daten von der übergeordneten Komponente (App.vue)
  props: {
    kunden:         { type: Array, default: () => [] },
    abos:           { type: Array, default: () => [] },
    ermaessigungen: { type: Array, default: () => [] },
    konten:         { type: Array, default: () => [] },
    banken:         { type: Array, default: () => [] }
  },

  emits: ['kunden-updated', 'show-toast'],

  data() {
    return {
      suchbegriff:  '',
      filterAboNr:  '',
      showModal:    false,
      editKunde:    null,
      sortBy:       'kundennr',
      sortOrder:    'asc',

      // Formularfelder – verwenden camelCase wie die API
      form: {
        kundennr: '',
        vorname:  '',
        nachname: '',
        iban:     '',
        bic:      '',
        bankName: '',
        abonr:    0,
        ermid:    0
      }
    }
  },

  computed: {
    // Filtert die Kundenliste nach Suchbegriff und Abo-Filter
    gefilterteKunden() {
      const gefiltert = this.kunden.filter(k => {
        const q = this.suchbegriff.toLowerCase()
        const matchSearch = !q ||
          k.vorname?.toLowerCase().includes(q)  ||
          k.nachname?.toLowerCase().includes(q) ||
          String(k.kundennr).includes(q)        ||
          k.iban?.toLowerCase().replace(/\s/g,'').includes(q.replace(/\s/g,'')) ||
          k.konto?.bic?.toLowerCase().includes(q) ||
          k.konto?.bank?.bankName?.toLowerCase().includes(q)

        // Abo-Filter: Prüft ob der Kunde ein entsprechendes Abo hat (über activeAbo oder direkt)
        const matchAbo = !this.filterAboNr
          || k.abonr === this.filterAboNr
          || k.aktivesAbo?.abonr === this.filterAboNr

        return matchSearch && matchAbo
      })

      // Sortieren
      return gefiltert.sort((a, b) => {
        let valA, valB

        if (this.sortBy === 'kundennr') {
          valA = a.kundennr
          valB = b.kundennr
        } else if (this.sortBy === 'vorname') {
          valA = a.vorname || ''
          valB = b.vorname || ''
        } else if (this.sortBy === 'nachname') {
          valA = a.nachname || ''
          valB = b.nachname || ''
        } else if (this.sortBy === 'iban') {
          valA = a.iban || a.konto?.iban || ''
          valB = b.iban || b.konto?.iban || ''
        } else if (this.sortBy === 'bic') {
          valA = a.konto?.bic || ''
          valB = b.konto?.bic || ''
        } else if (this.sortBy === 'bank') {
          valA = a.konto?.bank?.bankName || ''
          valB = b.konto?.bank?.bankName || ''
        } else if (this.sortBy === 'ermaessigung') {
          valA = a.ermaessigung?.ermaessigungssatz || 0
          valB = b.ermaessigung?.ermaessigungssatz || 0
        }

        if (valA === undefined) valA = ''
        if (valB === undefined) valB = ''

        let compare = 0
        if (typeof valA === 'number' && typeof valB === 'number') {
          compare = valA - valB
        } else {
          compare = String(valA).localeCompare(String(valB))
        }

        return this.sortOrder === 'asc' ? compare : -compare
      })
    },

    // Kunden mit mindestens einem verknüpften Abo (über KundenAbo)
    // Vereinfacht: zählt alle Kunden (vollständige Implementierung über KundenAbo-Endpunkt)
    aktivKunden()      { return this.kunden.length },
    ermaessigtKunden() { return this.kunden.filter(k => k.ermaessigung?.ermaessigungssatz > 0).length },
    ohneAbo()          { return 0 } // Würde einen JOIN über KundenAbo benötigen
  },

  methods: {
    // Öffnet das Modal im Neuanlage- oder Bearbeitenmodus
    openModal(kunde) {
      this.editKunde = kunde
      if (kunde) {
        // Vorhandene Daten ins Formular laden
        this.form = {
          kundennr: kunde.kundennr,
          vorname:  kunde.vorname  || '',
          nachname: kunde.nachname || '',
          iban:     kunde.iban || kunde.konto?.iban || '',
          bic:      kunde.konto?.bic || '',
          bankName: kunde.konto?.bank?.bankName || '',
          abonr:    kunde.abonr    || (kunde.aktivesAbo?.abonr || 0),
          ermid:    kunde.ermid    || (kunde.ermaessigung?.ermid || 0)
        }
      } else {
        // Leeres Formular für Neuanlage
        this.form = {
          kundennr: '',
          vorname: '',
          nachname: '',
          iban: '',
          bic: '',
          bankName: '',
          abonr: 0,
          ermid: 0
        }
      }
      this.showModal = true
    },

    closeModal() {
      this.showModal = false
      this.editKunde = null
    },

    sortiereNach(feld) {
      if (this.sortBy === feld) {
        this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc'
      } else {
        this.sortBy = feld
        this.sortOrder = 'asc'
      }
    },

    onBicInput() {
      this.form.bic = (this.form.bic || '').toUpperCase().trim();
      const gefundeneBank = this.banken.find(b => b.bic === this.form.bic);
      if (gefundeneBank) {
        this.form.bankName = gefundeneBank.bankName;
      }
    },

    getErmaessigungName(ermid) {
      const names = {
        1: 'Student',
        2: 'Keine Ermäßigung',
        3: 'Senior',
        4: 'Firmenrabatt',
        5: 'Jugend/Schüler',
        6: 'Sonderaktion'
      };
      return names[ermid] || `Ermäßigung #${ermid}`;
    },

    /* ── speichereKunde() ─────────────────────────────────────────────
       Sendet POST (Neuanlage) oder PUT (Bearbeiten) an die C# API.
       Ablauf:
       1. Konto anlegen/prüfen (POST /api/konten)
       2. Kunde anlegen/aktualisieren (POST oder PUT /api/kunden)
       3. Kundenliste neu laden
    ──────────────────────────────────────────────────────────────── */
    async speichereKunde() {
      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'

      // IBAN bereinigen (Leerzeichen entfernen) und in Großbuchstaben umwandeln
      const cleanIban = (this.form.iban || '').replace(/\s/g, '').toUpperCase()
      
      // Validierung: Eine deutsche/Standard-IBAN in diesem Kontext muss exakt 22 Zeichen lang sein
      if (cleanIban.length !== 22) {
        this.$emit('show-toast', {
          text: 'Fehler: Eine gültige IBAN muss exakt 22 Zeichen lang sein.',
          type: 'danger'
        })
        return
      }

      // Bereinigte IBAN zurückschreiben, damit sie korrekt an die API übermittelt wird
      this.form.iban = cleanIban

      try {
        // Schritt 1: Konto anlegen falls es noch nicht existiert
        const kontoBody = { iban: this.form.iban, bic: this.form.bic }
        const kontoExists = await fetch(`${baseUrl}/konten/${encodeURIComponent(this.form.iban)}`)

        if (!kontoExists.ok) {
          // Konto existiert nicht → erst Bank anlegen, dann Konto
          const bankExists = await fetch(`${baseUrl}/banken/${encodeURIComponent(this.form.bic)}`)
          if (!bankExists.ok) {
            await fetch(`${baseUrl}/banken`, {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ bic: this.form.bic, bankName: this.form.bankName })
            })
          }
          await fetch(`${baseUrl}/konten`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(kontoBody)
          })
        }

        // Schritt 2: Kunde anlegen oder aktualisieren
        // Das API erwartet: { vorname, nachname, iban, abonr, ermid }
        const kundeBody = {
          kundennr: this.form.kundennr || 0,
          vorname:  this.form.vorname,
          nachname: this.form.nachname,
          iban:     this.form.iban,
          abonr:    this.form.abonr || null,
          ermid:    this.form.ermid || null
        }

        const url    = this.editKunde ? `${baseUrl}/kunden/${this.form.kundennr}` : `${baseUrl}/kunden`
        const method = this.editKunde ? 'PUT' : 'POST'

        const res = await fetch(url, {
          method,
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(kundeBody)
        })

        if (res.ok) {
          // Schritt 3: Kundenliste neu laden (enthält echte KundenNr + Navigation)
          const resKunden = await fetch(`${baseUrl}/kunden`)
          if (resKunden.ok) {
            this.$emit('kunden-updated', await resKunden.json())
          }
          this.$emit('show-toast', {
            text: `Kunde ${this.form.vorname} ${this.form.nachname} erfolgreich gespeichert.`,
            type: 'success'
          })
        } else {
          const fehler = await res.text()
          this.$emit('show-toast', { text: `Fehler beim Speichern: ${fehler}`, type: 'danger' })
        }
      } catch (err) {
        // Offline-Fallback
        console.warn('API offline – lokale Änderung.', err)
        let neueKunden = [...this.kunden]
        if (this.editKunde) {
          const idx = neueKunden.findIndex(k => k.kundennr === this.form.kundennr)
          if (idx !== -1) neueKunden[idx] = {
            ...neueKunden[idx],
            vorname: this.form.vorname,
            nachname: this.form.nachname,
            iban: this.form.iban
          }
        } else {
          neueKunden.push({ ...this.form, kundennr: 'TEMP-' + Date.now() })
        }
        this.$emit('kunden-updated', neueKunden)
        this.$emit('show-toast', { text: 'API offline. Lokal gespeichert.', type: 'warning' })
      }

      this.closeModal()
    },

    // Löscht einen Kunden per DELETE-Request
    async loescheKunde(nr) {
      if (!confirm(`Kunden #${nr} wirklich löschen?`)) return

      // Sofort lokal entfernen (Optimistic UI)
      const neueKunden = this.kunden.filter(k => k.kundennr !== nr)
      this.$emit('kunden-updated', neueKunden)

      const baseUrl = localStorage.getItem('apiUrl') || 'http://localhost:5000/api'
      try {
        const res = await fetch(`${baseUrl}/kunden/${nr}`, { method: 'DELETE' })
        if (res.ok) {
          this.$emit('show-toast', { text: `Kunde #${nr} gelöscht.`, type: 'success' })
        } else {
          this.$emit('show-toast', { text: `API-Fehler beim Löschen: ${res.status}`, type: 'danger' })
        }
      } catch (err) {
        this.$emit('show-toast', { text: `Kunde #${nr} lokal gelöscht (API offline).`, type: 'warning' })
      }
    },

    // CSV-Export der aktuellen Kundenliste (berücksichtigt Filter)
    exportKunden() {
      const zeilen = ['KundenNr;Vorname;Nachname;IBAN;BIC;Bank']
      const exportList = this.gefilterteKunden || this.kunden
      exportList.forEach(k => {
        zeilen.push(`${k.kundennr};${k.vorname};${k.nachname};${k.iban || ''};${k.konto?.bic || ''};${k.konto?.bank?.bankName || ''}`)
      })
      const blob = new Blob([zeilen.join('\n')], { type: 'text/csv;charset=utf-8;' })
      const url  = URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href     = url
      link.download = 'kunden_export.csv'
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      URL.revokeObjectURL(url)
    },

    // Maskiert IBAN für Datenschutz: DE89...3000 → "DE89 **** **** **** 3000"
    ibanMask(iban) {
      if (!iban) return '—'
      const clean = iban.replace(/\s/g, '')
      return clean.slice(0, 4) + ' **** **** **** ' + clean.slice(-4)
    },

    // Formatiert Dezimalzahl als Preis: 49.99 → "49,99"
    formatPreis(p) {
      return parseFloat(p || 0).toFixed(2).replace('.', ',')
    }
  }
}
</script>
