<script setup>
import { ref, onMounted } from 'vue'

const sources = ref([])
const loading = ref(true)
const error = ref(null)

onMounted(async () => {
  try {
    // Note: The build script copies public/* (including index.json from aidoku build) into the root.
    const res = await fetch('./index.json')
    if (!res.ok) {
      throw new Error('Failed to load source list')
    }
    const data = await res.json()
    // Depending on aidoku build's output, it usually has an array of sources
    if (Array.isArray(data)) {
      sources.value = data
    } else if (data.sources) {
      sources.value = data.sources
    } else {
      sources.value = [data] // Fallback
    }
  } catch (err) {
    error.value = err.message
    console.error('Error fetching sourcelist:', err)
  } finally {
    loading.value = false
  }
})

const getListUrl = () => {
  // We need the absolute URL to the hosted index.min.json for the aidoku app
  const base = window.location.href.replace(/\/$/, '')
  return `aidoku://addSourceList?url=${base}/index.min.json`
}
</script>

<template>
  <header class="hero">
    <div class="hero-content">
      <img src="https://raw.githubusercontent.com/Aidoku/Aidoku/main/Aidoku/Assets.xcassets/AppIcon.appiconset/AppIcon-1024x1024.png" alt="Aidoku Logo" class="logo" />
      <h1>kunihir0 Sources</h1>
      <p class="subtitle">A repository of high-quality sources tailored for Aidoku.</p>
    </div>
    <div class="hero-actions">
      <a :href="getListUrl()" class="global-action-btn">
        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14"></path><path d="M5 12h14"></path></svg>
        Add Repository to Aidoku
      </a>
    </div>
  </header>

  <main>
    <div v-if="loading" class="state-container">
      <div class="loader-spinner"></div>
      <p>Loading Sources...</p>
    </div>

    <div v-else-if="error" class="state-container error">
      <svg xmlns="http://www.w3.org/-2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="12"></line><line x1="12" y1="16" x2="12.01" y2="16"></line></svg>
      <h2>Oops, something went wrong.</h2>
      <p>{{ error }}</p>
      <p class="help-text">Please check back later or make sure the GitHub Action has built the index.json properly.</p>
    </div>

    <div v-else-if="sources.length === 0" class="state-container">
      <h2>No Sources Found</h2>
      <p>The index.json is empty or no packages are available.</p>
    </div>

    <div v-else class="source-grid">
      <div v-for="source in sources" :key="source.id" class="source-card group">
        <div class="card-header">
          <div class="icon-placeholder">
            <img v-if="source.icon" :src="source.icon" alt="icon" @error="$event.target.style.display='none'" />
            <span v-else>{{ source.name ? source.name.charAt(0).toUpperCase() : '?' }}</span>
          </div>
          <div class="header-info">
            <h2 class="source-name">{{ source.name || 'Unknown Source' }}</h2>
            <div class="badges">
              <span class="badge version">v{{ source.version || '1.0' }}</span>
              <span class="badge lang" v-if="source.lang">{{ source.lang.toUpperCase() }}</span>
            </div>
          </div>
        </div>
        
        <p class="description">{{ source.description || 'No description provided.' }}</p>
      </div>
    </div>
  </main>

  <footer>
    <p>Powered by <a href="https://github.com/Aidoku/Aidoku" target="_blank" rel="noopener">Aidoku</a> • Generated via GitHub Actions</p>
  </footer>
</template>

<style scoped>
.hero {
  text-align: center;
  padding: 4rem 1rem;
  margin-bottom: 2rem;
}

.hero-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.hero-actions {
  display: flex;
  justify-content: center;
}

.global-action-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem 2rem;
  background: var(--accent-color);
  color: #ffffff;
  font-size: 1.1rem;
  font-weight: 700;
  border-radius: 9999px;
  box-shadow: 0 10px 20px rgba(99, 102, 241, 0.3);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.global-action-btn:hover {
  transform: translateY(-2px);
  background: var(--accent-hover);
  box-shadow: 0 15px 25px rgba(99, 102, 241, 0.4);
  color: #ffffff;
}

.logo {
  width: 100px;
  height: 100px;
  border-radius: 24px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.3);
  animation: float 6s ease-in-out infinite;
}

@keyframes float {
  0% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
  100% { transform: translateY(0px); }
}

h1 {
  font-size: 3rem;
  font-weight: 800;
  background: linear-gradient(135deg, #ffffff 0%, #a5b4fc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: -1px;
}

.subtitle {
  font-size: 1.25rem;
  color: var(--text-secondary);
  max-width: 600px;
}

.state-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem;
  text-align: center;
  background: var(--glass-bg);
  backdrop-filter: blur(12px);
  border: 1px solid var(--glass-border);
  border-radius: 24px;
  margin: 2rem 0;
  gap: 1rem;
}

.error svg {
  color: #ef4444;
  margin-bottom: 1rem;
}

.help-text {
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-top: 0.5rem;
}

.loader-spinner {
  width: 48px;
  height: 48px;
  border: 4px solid var(--glass-border);
  border-top-color: var(--accent-color);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.source-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
  gap: 2rem;
  padding-bottom: 4rem;
}

.source-card {
  background: var(--glass-bg);
  backdrop-filter: blur(12px);
  border: 1px solid var(--glass-border);
  border-radius: 20px;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.source-card:hover {
  transform: translateY(-4px);
  border-color: var(--accent-color);
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.icon-placeholder {
  width: 56px;
  height: 56px;
  border-radius: 14px;
  background: linear-gradient(135deg, var(--bg-secondary) 0%, var(--bg-primary) 100%);
  border: 1px solid var(--glass-border);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--text-secondary);
  overflow: hidden;
  flex-shrink: 0;
}

.icon-placeholder img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.header-info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.source-name {
  font-size: 1.25rem;
  margin: 0;
  color: var(--text-primary);
}

.badges {
  display: flex;
  gap: 0.5rem;
}

.badge {
  font-size: 0.75rem;
  padding: 0.2rem 0.6rem;
  border-radius: 9999px;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.badge.version {
  background: rgba(99, 102, 241, 0.1);
  color: #a5b4fc;
}

.badge.lang {
  background: rgba(16, 185, 129, 0.1);
  color: #6ee7b7;
}

.description {
  color: var(--text-secondary);
  font-size: 0.95rem;
  flex-grow: 1;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  -webkit-box-orient: vertical;  
  overflow: hidden;
  margin-bottom: 0.5rem;
}

footer {
  margin-top: auto;
  text-align: center;
  padding: 2rem 0;
  color: var(--text-secondary);
  font-size: 0.9rem;
  border-top: 1px solid var(--glass-border);
}

@media (max-width: 640px) {
  h1 { font-size: 2.25rem; }
  .source-grid { grid-template-columns: 1fr; }
}
</style>
