<script setup>
import { ref, onMounted, computed } from 'vue'

const paises = ref([])
const busqueda = ref('')
const cargando = ref(true)
const errorCarga = ref('')
const regionSeleccionada = ref('')

const paisesFiltrados = computed(() => {
  const textoBusqueda = busqueda.value
    .toLocaleLowerCase('es')
    .trim()

  return paises.value
    .filter(pais => {
      const nombre =
        pais.names.translations?.spa?.common ||
        pais.names.common ||
        ''

      const coincideNombre = nombre
        .toLocaleLowerCase('es')
        .includes(textoBusqueda)

      const coincideRegion =
        !regionSeleccionada.value ||
        pais.region === regionSeleccionada.value

      return coincideNombre && coincideRegion
    })
    .sort((a, b) => {
      const nombreA =
        a.names.translations?.spa?.common ||
        a.names.common ||
        ''

      const nombreB =
        b.names.translations?.spa?.common ||
        b.names.common ||
        ''

      return nombreA.localeCompare(nombreB, 'es', {
        sensitivity: 'base'
      })
    })
})

function traducirRegion(region) {
  const regiones = {
    Africa: 'África',
    Americas: 'América',
    Asia: 'Asia',
    Europe: 'Europa',
    Oceania: 'Oceanía',
    Antarctic: 'Antártida'
  }

  return regiones[region] || region || 'No disponible'
}

onMounted(async () => {
  cargando.value = true
  errorCarga.value = ''

  try {
    const respuesta = await fetch(
      'https://api.restcountries.com/countries/v5?limit=100',
      {
        headers: {
          Authorization: 'rc_live_b20ec7e6b3f94a1199bf168ba11ebd26'
        }
      }
    )
      
    if (!respuesta.ok) {
      throw new Error(`La API respondió HTTP ${respuesta.status}`)
    }

    const resultado = await respuesta.json()

    if (!Array.isArray(resultado.data?.objects)) {
      throw new Error('La respuesta de la API no tiene el formato esperado.')
    }

    paises.value = resultado.data.objects
  } catch (error) {
    console.error('Error al consultar los países:', error)
    errorCarga.value =
      'No se pudieron cargar los países. Verifica tu conexión e inténtalo de nuevo.'
  } finally {
    cargando.value = false
  }
})
</script>

<template>
  <div class="pagina">
    <header class="encabezado">
      <a class="logo" href="#">
        <span class="logo-icono">🌎</span>
        <span>Country Explorer</span>
      </a>

      <span class="etiqueta">Explora el mundo</span>
    </header>

    <main class="contenido">
      <section class="presentacion">
        <span class="subtitulo">DESCUBRE EL MUNDO</span>

        <h1>Explorador de Países</h1>

        <p>
          Conoce países de todo el mundo, sus capitales,
          regiones y datos interesantes.
        </p>

        <div class="buscador">
          <span class="buscador-icono">⌕</span>
          <input
            v-model="busqueda"
            type="text"
            placeholder="Buscar un país..."
            aria-label="Buscar un país"
          />
        </div>

        <div class="filtro-region">
          <label for="region">Filtrar por región:</label>

          <select id="region" v-model="regionSeleccionada">
            <option value="">Todas las regiones</option>
            <option value="Africa">África</option>
            <option value="Americas">América</option>
            <option value="Asia">Asia</option>
            <option value="Europe">Europa</option>
            <option value="Oceania">Oceanía</option>
            <option value="Antarctic">Antártida</option>
          </select>
        </div>
      </section>

      <section class="seccion-paises">
        <div class="seccion-titulo">
          <div>
            <h2>Países del mundo</h2>
            <p>Información obtenida desde REST Countries.</p>
          </div>

          <span class="contador">
            {{ paisesFiltrados.length }} países
          </span>
        </div>

        <p v-if="cargando" class="mensaje-estado">
          Cargando países...
        </p>

        <p v-else-if="errorCarga" class="mensaje-estado error">
          {{ errorCarga }}
        </p>

        <p v-else-if="paisesFiltrados.length === 0" class="mensaje-estado">
          No se encontraron países que coincidan con tu búsqueda.
        </p>

        <div v-else class="grid-paises">
          <article
            v-for="pais in paisesFiltrados"
            :key="pais.codes.alpha_3"
            class="tarjeta-pais"
          >
            <div class="bandera">
              <img
                v-if="pais.flag?.url_png"
                :src="pais.flag.url_png"
                :alt="`Bandera de ${
                  pais.names.translations?.spa?.common ||
                  pais.names.common
                }`"
                class="imagen-bandera"
              />
              <span v-else>🏳️</span>
            </div>

            <div class="datos-pais">
              <span class="codigo-pais">
                {{ pais.codes.alpha_2 }}
              </span>

              <h3>
                {{ pais.names.translations?.spa?.common || pais.names.common }}
              </h3>

              <p class="nombre-oficial">
                {{ pais.names.translations?.spa?.official || pais.names.official }}
              </p>

              <div class="detalle">
                <span>Capital</span>
                <strong>
                  {{ pais.capitals?.[0]?.name || 'No disponible' }}
                </strong>
              </div>

              <div class="detalle">
                <span>Región</span>
                <strong>{{ traducirRegion(pais.region) }}</strong>
              </div>

              <div class="detalle">
                <span>Población</span>
                <strong>
                  {{ pais.population?.toLocaleString('es-MX') ?? 'No disponible' }}
                </strong>
              </div>
            </div>
          </article>
        </div>
      </section>

      <footer class="pie">
        <span>🌐 Explorador de Países</span>
        <span>Proyecto Unidad 3</span>
      </footer>
    </main>
  </div>
</template>