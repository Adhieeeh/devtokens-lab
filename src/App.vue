<script setup>
import { ref, computed } from 'vue'


const issuerClaim = ref('https://api.adhieeeh.dev')
const subjectClaim = ref('usr_dev_77a29b')
const tokenLifespan = ref(3600) 
const chosenAlgorithm = ref('HS256')
const tokenSecretKey = ref('super-secret-dev-studio-key')


const customMetadata = ref([
  { key: 'role', value: 'admin' },
  { key: 'tier', value: 'premium' }
])


const addMetaField = () => {
  customMetadata.value.push({ key: '', value: '' })
}
const removeMetaField = (index) => {
  customMetadata.value.splice(index, 1)
}


const compiledTokenStructure = computed(() => {

  const basePayload = {
    iss: issuerClaim.value || 'undefined',
    sub: subjectClaim.value || 'anonymous',
    exp: Math.floor(Date.now() / 1000) + tokenLifespan.value,
    iat: Math.floor(Date.now() / 1000)
  }


  customMetadata.value.forEach(meta => {
    if (meta.key.trim()) {
      basePayload[meta.key.trim()] = meta.value
    }
  })

 
  const headerMock = btoa(JSON.stringify({ alg: chosenAlgorithm.value, typ: 'JWT' })).replace(/=/g, '')
  const payloadMock = btoa(JSON.stringify(basePayload)).replace(/=/g, '')
  const signatureMock = btoa(`sha256(${headerMock}.${payloadMock}, ${tokenSecretKey.value})`).substring(0, 32).replace(/=/g, '')

  return `${headerMock}.${payloadMock}.${signatureMock}`
})


const copyStatus = ref('📋 Copy Compiled JWT')
const copyTokenToClipboard = () => {
  navigator.clipboard.writeText(compiledTokenStructure.value)
  copyStatus.value = 'Token Copied! ⚡'
  setTimeout(() => {
    copyStatus.value = '📋 Copy Compiled JWT'
  }, 2000)
}
</script>

<template>
  <div class="app-wrapper">
    
    <header class="app-header">
      <h1> DevTokens Laboratory</h1>
      <p>An interactive Vue 3 client sandbox designed to map out, serialize, and test JSON Web Token payload matrices.</p>
    </header>

    <main class="studio-desk">
      
      <section class="control-panel">
        <h3>Token Claims Configuration</h3>
        
        <div class="control-group">
          <label>Issuer Claim (iss)</label>
          <input type="text" v-model="issuerClaim">
        </div>

        <div class="control-group">
          <label>Subject Claim (sub)</label>
          <input type="text" v-model="subjectClaim">
        </div>

        <div class="control-group">
          <label>Token Lifespan / TTL ({{ tokenLifespan }}s)</label>
          <input type="range" min="60" max="86400" step="60" v-model.number="tokenLifespan">
        </div>

        <div class="flex-row gap-15">
          <div class="control-group flex-1">
            <label>Hashing Algorithm</label>
            <select v-model="chosenAlgorithm">
              <option value="HS256">HS256 (HMAC-SHA256)</option>
              <option value="RS256">RS256 (RSA-SHA256)</option>
              <option value="ES256">ES256 (ECDSA-SHA256)</option>
            </select>
          </div>
          <div class="control-group flex-1">
            <label>Signing Secret Key</label>
            <input type="password" v-model="tokenSecretKey" class="secret-input">
          </div>
        </div>

        <div class="meta-section">
          <div class="meta-header">
            <label>Injected Scope Claims (Claims Matrix)</label>
            <button type="button" @click="addMetaField" class="add-btn">+ Add Field</button>
          </div>
          
          <div v-for="(meta, index) in customMetadata" :key="index" class="meta-row">
            <input type="text" v-model="meta.key" placeholder="claim key" class="meta-input">
            <input type="text" v-model="meta.value" placeholder="value" class="meta-input">
            <button type="button" @click="removeMetaField(index)" class="del-btn">✕</button>
          </div>
        </div>
      </section>

      <section class="output-panel">
        <div class="output-header-row">
          <h3>Calculated Encrypted Token String</h3>
          <button @click="copyTokenToClipboard" class="copy-btn">{{ copyStatus }}</button>
        </div>

        <div class="jwt-terminal-wrapper">
          <div class="terminal-body">
            <code>{{ compiledTokenStructure }}</code>
          </div>
          <div class="terminal-footer">
            <span class="indicator-dot red"></span><span>Header</span>
            <span class="indicator-dot green"></span><span>Payload</span>
            <span class="indicator-dot blue"></span><span>Signature</span>
          </div>
        </div>
      </section>

    </main>
  </div>
</template>

<style scoped>
.app-wrapper {
  max-width: 1200px;
  margin: 40px auto;
  padding: 0 24px;
  font-family: system-ui, -apple-system, sans-serif;
  color: #f8fafc;
}

.app-header {
  border-bottom: 2px solid #1e293b;
  padding-bottom: 20px;
  margin-bottom: 35px;
}

.app-header h1 {
  margin: 0;
  font-size: 30px;
  color: #f43f5e;
  letter-spacing: -0.5px;
}

.app-header p {
  margin: 4px 0 0 0;
  color: #64748b;
  font-size: 14px;
}

.studio-desk {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(360px, 1fr));
  gap: 40px;
}

.control-panel {
  background-color: #0f172a;
  border: 1px solid #1e293b;
  padding: 25px;
  border-radius: 16px;
  box-shadow: 0 10px 15px -3px rgba(0,0,0,0.3);
}

.control-panel h3 {
  margin-top: 0;
  margin-bottom: 20px;
  font-size: 14px;
  color: #64748b;
  text-transform: uppercase;
}

.control-group {
  margin-bottom: 18px;
}

.control-group label {
  display: block;
  font-size: 12px;
  font-weight: 700;
  color: #cbd5e1;
  margin-bottom: 6px;
}

.control-panel input[type="text"],
.control-panel input[type="password"],
.control-panel select {
  width: 100%;
  padding: 10px 14px;
  background-color: #020617;
  border: 1px solid #1e293b;
  border-radius: 8px;
  color: #fff;
  font-size: 14px;
  box-sizing: border-box;
}

.secret-input {
  font-family: monospace;
  color: #fbbf24 !important;
}

.control-panel input[type="range"] {
  width: 100%;
  accent-color: #f43f5e;
  cursor: pointer;
}

.flex-row { display: flex; }
.gap-15 { gap: 15px; }
.flex-1 { flex: 1; }

.meta-section {
  margin-top: 25px;
  border-top: 1px solid #1e293b;
  padding-top: 18px;
}

.meta-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.meta-header label {
  font-size: 12px;
  font-weight: 700;
  color: #cbd5e1;
}

.add-btn {
  background: none;
  border: 1px dashed #f43f5e;
  color: #f43f5e;
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 11px;
  font-weight: 700;
  cursor: pointer;
}

.meta-row {
  display: flex;
  gap: 10px;
  margin-bottom: 8px;
  align-items: center;
}

.meta-input {
  flex: 1;
  padding: 8px !important;
  background-color: #020617;
  border: 1px solid #1e293b;
  border-radius: 6px;
  color: #fff;
  font-size: 13px;
}

.del-btn {
  background: none;
  border: none;
  color: #64748b;
  cursor: pointer;
  font-size: 14px;
}

.del-btn:hover { color: #ef4444; }

.output-panel {
  display: flex;
  flex-direction: column;
}

.output-header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.output-header-row h3 {
  margin: 0;
  font-size: 14px;
  color: #64748b;
  text-transform: uppercase;
}

.copy-btn {
  padding: 8px 16px;
  background-color: transparent;
  border: 1px solid #f43f5e;
  color: #f43f5e;
  border-radius: 8px;
  font-weight: 700;
  font-size: 13px;
  cursor: pointer;
  transition: 0.2s;
}

.copy-btn:hover {
  background-color: #f43f5e;
  color: #020617;
}

.jwt-terminal-wrapper {
  background-color: #020617;
  border: 1px solid #1e293b;
  border-radius: 16px;
  padding: 25px;
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-height: 250px;
}

.terminal-body {
  font-family: monospace;
  font-size: 14px;
  line-height: 1.6;
  color: #f43f5e;
  word-break: break-all;
}

.terminal-footer {
  margin-top: 20px;
  border-top: 1px solid #1e293b;
  padding-top: 15px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 11px;
  color: #64748b;
  font-weight: 700;
  text-transform: uppercase;
}

.indicator-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  display: inline-block;
  margin-left: 10px;
}
.indicator-dot.red { background-color: #f43f5e; margin-left: 0; }
.indicator-dot.green { background-color: #10b981; }
.indicator-dot.blue { background-color: #38bdf8; }
</style>