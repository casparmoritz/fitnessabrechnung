<template>
  <!-- ==========================================
    EinstellungenView.vue – Systeminformationen & API-Konfiguration
    ========================================== -->
  <div class="page">
    <div class="page-header">
      <div>
        <h2 class="page-title">Einstellungen</h2>
        <p class="page-desc">Systeminformationen und API-Konfiguration</p>
      </div>
    </div>

    <div class="grid-2">

      <!-- System-Info -->
      <div class="card">
        <div class="card-header">
          <div class="card-title">Systeminformationen</div>
        </div>
        <div class="info-list">
          <div class="info-row"><span>Anwendung</span><span>FitnessAbrechnung</span></div>
          <div class="info-row"><span>Version</span><span class="badge badge-primary">v1.0.0</span></div>
          <div class="info-row"><span>Auftraggeber</span><span>Fitti Exempel GmbH</span></div>
          <div class="info-row"><span>Auftragnehmer</span><span>JCR IT Solution GmbH</span></div>
          <div class="info-row"><span>Pflichtenheft</span><span class="td-muted">Version 1.2</span></div>
          <div class="info-row"><span>Frontend</span><span class="badge badge-accent">Vue.js 3</span></div>
          <div class="info-row"><span>Backend</span><span class="badge badge-neutral">C# REST-API</span></div>
          <div class="info-row"><span>Datenbank</span><span class="badge badge-neutral">Oracle DB</span></div>
        </div>
      </div>

      <!-- API-Konfiguration -->
      <div class="card">
        <div class="card-header">
          <div class="card-title">API-Konfiguration</div>
        </div>
        <div class="form-group mb-lg">
          <label for="input-api-url">API Base-URL</label>
          <input id="input-api-url" type="text" v-model="apiUrl" placeholder="http://localhost:5000/api" />
        </div>
        <div class="flex gap-sm items-center mb-lg">
          <div class="status-dot" :class="apiStatus === 'online' ? 'dot-green' : 'dot-red'"></div>
          <span style="font-size:13px;">
            API Status:
            <strong>{{ apiStatus === 'online' ? 'Verbunden' : 'Nicht erreichbar' }}</strong>
          </span>
        </div>
        <button class="btn btn-ghost w-full" id="btn-api-test" @click="testApiVerbindung">
          🔗 Verbindung testen
        </button>
      </div>

    </div>
  </div>
</template>

<script>
/* =========================================================
   JAVASCRIPT – EinstellungenView.vue
   Systeminfo und API-Verbindungstest
   ========================================================= */

export default {
  name: 'EinstellungenView',

  // data() = alle reaktiven Variablen dieser Komponente
  data() {
    return {
      // Die API-URL die im Textfeld angezeigt und bearbeitbar ist
      apiUrl: 'http://localhost:5000/api',

      // Aktueller Verbindungsstatus: 'online' oder 'offline'
      apiStatus: 'offline'
    }
  },

  methods: {
    // ✅ IMPLEMENTIERT: API-Verbindung testen (GET /api/health)
    async testApiVerbindung() {
      try {
        // Sendet einen GET-Request an den Health-Check-Endpunkt des Backends
        const res = await fetch(this.apiUrl + '/health')
        // Wenn der Server antwortet und der Statuscode OK ist → online
        this.apiStatus = res.ok ? 'online' : 'offline'
      } catch {
        // Fehler (z.B. Server nicht erreichbar) → offline
        this.apiStatus = 'offline'
      }
    }
  }
}
</script>

<style scoped>
.info-list { display: flex; flex-direction: column; gap: var(--sp-sm); }
.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid var(--clr-border);
  font-size: 14px;
}
.info-row:last-child { border-bottom: none; }
.info-row > span:first-child { color: var(--clr-text-muted); }
.info-row > span:last-child  { font-weight: 600; }

.status-dot { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
.dot-green  { background: var(--clr-success); box-shadow: 0 0 8px var(--clr-success); }
.dot-red    { background: var(--clr-danger);  box-shadow: 0 0 8px var(--clr-danger); }
</style>
