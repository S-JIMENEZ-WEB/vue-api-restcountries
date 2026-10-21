<script setup>
import { computed, onMounted, ref } from 'vue'

// API REST Countries v5. El endpoint de listado permite filtrar después en Vue.
// La clave rc_live_demo es de demostración y no sustituye una clave personal.
const API_BASE = 'https://atlas-paises-api.edsonsam16072006.workers.dev/'
const PAGE_SIZE = 100

const paises = ref([])
const cargando = ref(true)
const error = ref('')
const busqueda = ref('')
const regionSeleccionada = ref('Todas')
const ultimaActualizacion = ref('')

const regiones = computed(() => {
  const valores = paises.value
    .map((pais) => pais.region)
    .filter((region) => typeof region === 'string' && region.trim())
  return ['Todas', ...new Set(valores)].sort((a, b) => {
    if (a === 'Todas') return -1
    if (b === 'Todas') return 1
    return a.localeCompare(b, 'es')
  })
})


const paisesFiltrados = computed(() => {
  const texto = busqueda.value.trim().toLocaleLowerCase('es')

  return paises.value.filter((pais) => {
    const nombreEspanol = nombrePais(pais).toLocaleLowerCase('es')
    const nombreIngles = (pais.names?.common || pais.name?.common || '').toLocaleLowerCase('es')
    const region = (pais.region || '').toLocaleLowerCase('es')

    const coincideTexto =
      !texto ||
      nombreEspanol.includes(texto) ||
      nombreIngles.includes(texto) ||
      region.includes(texto)

    const coincideRegion =
      regionSeleccionada.value === 'Todas' ||
      pais.region === regionSeleccionada.value

    return coincideTexto && coincideRegion
  })
})


const totalPaises = computed(() => paises.value.length)
const totalRegiones = computed(() => new Set(paises.value.map((p) => p.region).filter(Boolean)).size)
const poblacionTotal = computed(() => paises.value.reduce((total, pais) => total + (Number(pais.population) || 0), 0))

function nombrePais(pais) {
  return (
    pais.names?.translations?.spa?.common ||
    pais.translations?.spa?.common ||
    pais.names?.common ||
    pais.name?.common ||
    'Nombre no disponible'
  )
}

function capitalPais(pais) {
  if (Array.isArray(pais.capitals) && pais.capitals.length) {
    return pais.capitals.map((capital) => typeof capital === 'string' ? capital : capital.name).filter(Boolean).join(', ')
  }
  if (Array.isArray(pais.capital) && pais.capital.length) return pais.capital.join(', ')
  return 'Sin capital registrada'
}


function banderaPais(pais) {
  const codigo = pais.codes?.alpha_2 || pais.cca2

  if (!codigo || codigo.length !== 2) return '🌐'

  return [...codigo.toUpperCase()]
    .map(letra => String.fromCodePoint(127397 + letra.charCodeAt(0)))
    .join('')
}

function codigoPais(pais) {
  return pais.codes?.alpha_2 || pais.cca2 || '—'
}

function formatoNumero(valor) {
  if (valor === null || valor === undefined || valor === '' || !Number.isFinite(Number(valor))) return 'No disponible'
  return new Intl.NumberFormat('es-MX').format(Number(valor))
}

async function pedirPagina(offset) {
  const parametros = new URLSearchParams({
    limit: String(PAGE_SIZE),
    offset: String(offset),
    pretty: '1',
  })
  const respuesta = await fetch(`${API_BASE}?${parametros.toString()}`)
  const resultado = await respuesta.json()

 
if (!respuesta.ok || resultado.errors?.length || resultado.error) {
  const detalle = resultado.errors?.[0]?.message || resultado.error || `HTTP ${respuesta.status}`
  throw new Error(`La API respondió con un problema: ${detalle}`)
}


  const objetos = resultado.data?.objects
  if (!Array.isArray(objetos)) {
    throw new Error('La respuesta de la API no contiene la lista esperada de países.')
  }

  return {
    paises: objetos,
    total: Number(resultado.data?.meta?.total) || objetos.length,
    mas: Boolean(resultado.data?.meta?.more),
  }
}

async function cargarPaises() {
  cargando.value = true
  error.value = ''

  try {
    // La primera petición se ejecuta al montar el componente, mediante onMounted().
    const primeraPagina = await pedirPagina(0)
    const coleccion = [...primeraPagina.paises]
    const offsets = []

    // La API entrega hasta 100 registros por página en el plan gratuito.
    for (let offset = PAGE_SIZE; offset < primeraPagina.total; offset += PAGE_SIZE) {
      offsets.push(offset)
    }

    if (offsets.length) {
      const paginas = await Promise.all(offsets.map((offset) => pedirPagina(offset)))
      paginas.forEach((pagina) => coleccion.push(...pagina.paises))
    }

    // Elimina posibles repetidos usando código de país o nombre como respaldo.
    const unicos = new Map()
    coleccion.forEach((pais) => {
      const clave = codigoPais(pais) !== '—' ? codigoPais(pais) : nombrePais(pais)
      unicos.set(clave, pais)
    })
    paises.value = [...unicos.values()].sort((a, b) => nombrePais(a).localeCompare(nombrePais(b), 'es'))
    ultimaActualizacion.value = new Intl.DateTimeFormat('es-MX', {
      dateStyle: 'medium',
      timeStyle: 'short',
    }).format(new Date())
  } catch (problema) {
    console.error('Error al consultar REST Countries:', problema)
    error.value = `${problema.message || 'No fue posible obtener los datos.'} Revisa la conexión o la configuración de acceso de la API.`
  } finally {
    cargando.value = false
  }
}

function limpiarFiltros() {
  busqueda.value = ''
  regionSeleccionada.value = 'Todas'
}

onMounted(() => {
  cargarPaises()
})
</script>

<template>
  <div class="page-shell">
    <header class="topbar">
      <a class="brand" href="#inicio" aria-label="Atlas, inicio">
        <span class="brand-mark" aria-hidden="true">✦</span>
        <span>atlas<span class="brand-dot">.</span></span>
      </a>
      <span class="topbar-note"><span class="status-dot"></span> Datos conectados a REST Countries</span>
    </header>

    <main id="inicio">
      <section class="hero">
        <div class="hero-copy">
          <p class="eyebrow"><span class="eyebrow-line"></span> EXPLORA EL MUNDO</p>
          <h1>Un mundo de países.<br /><span>Todo en un solo lugar.</span></h1>
          <p class="hero-description">
            Descubre banderas, capitales, regiones y población. Busca un país o explora por continente con información obtenida desde una API.
          </p>
          <div class="hero-pills">
            <span><span class="pill-icon">⌁</span> Vue 3</span>
            <span><span class="pill-icon">↗</span> Fetch API</span>
            <span><span class="pill-icon">◎</span> Datos dinámicos</span>
          </div>
        </div>
        <div class="hero-art" aria-hidden="true">
          <div class="orbit orbit-one"></div>
          <div class="orbit orbit-two"></div>
          <div class="globe">🌎</div>
          <span class="float-flag flag-one">🇨🇦</span>
          <span class="float-flag flag-two">🇯🇵</span>
          <span class="float-flag flag-three">🇲🇽</span>
          <span class="float-star star-one">✦</span>
          <span class="float-star star-two">✧</span>
        </div>
      </section>

      <section class="stats" aria-label="Resumen de datos">
        <article class="stat-card">
          <span class="stat-icon">🌐</span>
          <div><span class="stat-label">Países cargados</span><strong>{{ cargando ? '...' : formatoNumero(totalPaises) }}</strong></div>
        </article>
        <article class="stat-card">
          <span class="stat-icon">🧭</span>
          <div><span class="stat-label">Regiones disponibles</span><strong>{{ cargando ? '...' : formatoNumero(totalRegiones) }}</strong></div>
        </article>
        <article class="stat-card">
          <span class="stat-icon">👥</span>
          <div><span class="stat-label">Población registrada</span><strong>{{ cargando ? '...' : formatoNumero(poblacionTotal) }}</strong></div>
        </article>
      </section>

      <section class="explorer-section">
        <div class="section-heading">
          <div>
            <p class="eyebrow section-eyebrow">DIRECTORIO GLOBAL</p>
            <h2>Explorador de países</h2>
            <p class="section-subtitle">Encuentra tu próximo destino o aprende algo nuevo.</p>
          </div>
          <div class="results-counter" v-if="!cargando && !error">
            <strong>{{ formatoNumero(paisesFiltrados.length) }}</strong>
            <span>{{ paisesFiltrados.length === 1 ? 'resultado' : 'resultados' }}</span>
          </div>
        </div>

        <div class="filters-panel">
          <label class="search-box">
            <span class="search-icon" aria-hidden="true">⌕</span>
            <input
              v-model="busqueda"
              type="search"
              aria-label="Buscar país por nombre o región"
              placeholder="Buscar país o región..."
            />
            <kbd>⌘ / Ctrl + K</kbd>
          </label>
          <label class="region-filter">
            <span>Región</span>
            <select v-model="regionSeleccionada" aria-label="Filtrar por región">
              <option v-for="region in regiones" v-bind:key="region" v-bind:value="region">{{ region }}</option>
            </select>
          </label>
          <button class="clear-button" type="button" @click="limpiarFiltros">Limpiar</button>
        </div>

        <div v-if="cargando" class="state-panel loading-panel" role="status" aria-live="polite">
          <span class="loader"></span>
          <h3>Cargando países...</h3>
          <p>Estamos consultando la información más reciente de REST Countries.</p>
        </div>

        <div v-else-if="error" class="state-panel error-panel" role="alert">
          <span class="state-illustration">⚠</span>
          <h3>No fue posible cargar los países</h3>
          <p>{{ error }}</p>
          <button class="primary-button" type="button" @click="cargarPaises">Intentar de nuevo <span>↻</span></button>
        </div>

        <div v-else-if="paisesFiltrados.length === 0" class="state-panel empty-panel">
          <span class="state-illustration">⌕</span>
          <h3>No encontramos resultados</h3>
          <p>Prueba con otro nombre o selecciona una región diferente.</p>
          <button class="primary-button" type="button" @click="limpiarFiltros">Ver todos los países</button>
        </div>

        <div v-else class="country-grid">
          <article v-for="pais in paisesFiltrados" v-bind:key="codigoPais(pais) + nombrePais(pais)" class="country-card">
            <div class="country-card-top">       
<img
  class="flag-emoji"
  :src="`https://flagcdn.com/w80/${codigoPais(pais).toLowerCase()}.png`"
  :alt="`Bandera de ${nombrePais(pais)}`"
  width="48"
  height="32"
  style="object-fit: contain;"
/>
<span classs="country-code">{{  codigoPais(pais) }}</span>
            </div>
            <h3>{{ nombrePais(pais) }}</h3>
            <span class="region-badge">{{ pais.region || 'Región no disponible' }}</span>
            <div class="country-details">
              <div class="detail-row">
                <span class="detail-icon">⌖</span>
                <span class="detail-label">Capital</span>
                <strong>{{ capitalPais(pais) }}</strong>
              </div>
              <div class="detail-row">
                <span class="detail-icon">◎</span>
                <span class="detail-label">Población</span>
                <strong>{{ formatoNumero(pais.population) }}</strong>
              </div>
            </div>
            <div class="card-footer"><span>Ver ficha del país</span><span class="arrow-circle">↗</span></div>
          </article>
        </div>

        <p v-if="!cargando && !error && ultimaActualizacion" class="updated-note">
          <span class="status-dot"></span> Datos consultados el {{ ultimaActualizacion }} · Fuente: REST Countries API
        </p>
      </section>
    </main>

    <footer class="footer">
      <a class="brand footer-brand" href="#inicio"><span class="brand-mark">✦</span><span>atlas<span class="brand-dot">.</span></span></a>
      <span>Proyecto académico · Consumo de API con Vue 3</span>
      <a href="https://restcountries.com/docs/countries" target="_blank" rel="noreferrer">Documentación de la API ↗</a>
    </footer>
  </div>
</template>
