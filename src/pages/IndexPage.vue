<template>
  <q-page class="q-pa-sm q-pa-md-md q-pa-lg-lg">
    <div class="text-h4 text-h5-sm text-weight-bold text-center q-mb-md">Digimon Gallery</div>

    <!-- Filtros -->
    <q-card class="q-mb-md">
      <q-card-section class="q-pa-sm q-pa-md-md">
        <div class="row q-col-gutter-sm q-col-gutter-md-md">
          <div class="col-12 col-md-6">
            <q-input
              v-model="searchName"
              outlined
              dense
              label="Buscar por nombre"
              clearable
              @update:model-value="filterDigimons"
            >
              <template v-slot:prepend>
                <q-icon name="search" />
              </template>
            </q-input>
          </div>
          <div class="col-12 col-md-6">
            <q-select
              v-model="selectedLevel"
              outlined
              dense
              label="Filtrar por nivel"
              :options="levelOptions"
              clearable
              @update:model-value="filterDigimons"
            >
              <template v-slot:prepend>
                <q-icon name="filter_list" />
              </template>
            </q-select>
          </div>
        </div>
      </q-card-section>
    </q-card>

    <!-- Loading -->
    <div v-if="loading" class="flex flex-center q-pa-xl">
      <q-spinner-dots color="primary" size="50px" />
    </div>

    <!-- Grid de Digimons -->
    <div v-else class="row q-col-gutter-sm q-col-gutter-md-md q-col-gutter-lg-lg">
      <div
        v-for="digimon in paginatedDigimons"
        :key="digimon.name"
        class="col-12 col-xs-6 col-sm-6 col-md-4 col-lg-3 col-xl-2"
      >
        <q-card class="digimon-card full-height" clickable @click="goToDetail(digimon.name)">
          <q-img :src="digimon.img" :alt="digimon.name" ratio="1" class="digimon-image">
            <template v-slot:loading>
              <q-spinner-gears color="primary" />
            </template>
          </q-img>

          <q-card-section class="q-pa-sm q-pa-md-md">
            <div class="text-subtitle1 text-body2-sm text-weight-bold ellipsis">
              {{ digimon.name }}
            </div>
            <div class="text-caption text-grey-7">{{ digimon.level }}</div>
          </q-card-section>

          <q-card-actions align="right" class="q-pa-sm">
            <q-btn flat dense color="primary" icon="visibility" size="sm">
              <q-tooltip>Ver detalles</q-tooltip>
            </q-btn>
          </q-card-actions>
        </q-card>
      </div>
    </div>

    <!-- Sin resultados -->
    <div v-if="!loading && filteredDigimons.length === 0" class="text-center q-pa-xl text-grey-7">
      <q-icon name="search_off" size="48px" class="q-mb-md" />
      <div class="text-h6 text-body1-sm">No se encontraron Digimones</div>
    </div>

    <!-- Paginación -->
    <div v-if="!loading && filteredDigimons.length > 0" class="flex flex-center q-mt-md q-mb-sm">
      <q-pagination
        v-model="currentPage"
        :max="totalPages"
        :max-pages="isMobile ? 5 : 7"
        direction-links
        boundary-links
        color="primary"
        active-color="primary"
        :size="isMobile ? 'sm' : 'md'"
        @update:model-value="onPageChange"
      />
    </div>

    <!-- Info de paginación -->
    <div
      v-if="!loading && filteredDigimons.length > 0"
      class="text-center q-mb-md text-grey-7 text-caption text-body2-md"
    >
      Mostrando {{ startIndex + 1 }} - {{ endIndex }} de {{ filteredDigimons.length }} Digimones
    </div>
  </q-page>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import { Notify, useQuasar } from 'quasar'

const $q = useQuasar()
const router = useRouter()
const digimons = ref([])
const filteredDigimons = ref([])
const loading = ref(false)
const searchName = ref('')
const selectedLevel = ref(null)
const levelOptions = ref([])

// Paginación
const currentPage = ref(1)
const itemsPerPage = ref(12)

// Detectar si es móvil
const isMobile = computed(() => $q.screen.lt.md)

const totalPages = computed(() => {
  return Math.ceil(filteredDigimons.value.length / itemsPerPage.value)
})

const startIndex = computed(() => {
  return (currentPage.value - 1) * itemsPerPage.value
})

const endIndex = computed(() => {
  const end = currentPage.value * itemsPerPage.value
  return end > filteredDigimons.value.length ? filteredDigimons.value.length : end
})

const paginatedDigimons = computed(() => {
  return filteredDigimons.value.slice(startIndex.value, endIndex.value)
})

const fetchDigimons = async () => {
  loading.value = true
  try {
    const response = await axios.get('https://digimon-api.vercel.app/api/digimon')
    digimons.value = response.data
    filteredDigimons.value = response.data

    // Extraer niveles únicos para el filtro
    const levels = [...new Set(response.data.map((d) => d.level))]
    levelOptions.value = levels.sort()
  } catch (error) {
    console.error('Error fetching digimons:', error)
    Notify.create({
      type: 'negative',
      message: 'Error al cargar los Digimons',
      position: 'top',
    })
  } finally {
    loading.value = false
  }
}

const filterDigimons = () => {
  let result = digimons.value

  // Filtrar por nombre
  if (searchName.value) {
    result = result.filter((d) => d.name.toLowerCase().includes(searchName.value.toLowerCase()))
  }

  // Filtrar por nivel
  if (selectedLevel.value) {
    result = result.filter((d) => d.level === selectedLevel.value)
  }

  filteredDigimons.value = result
  currentPage.value = 1 // Resetear a la primera página cuando se filtra
}

const onPageChange = () => {
  // Scroll hacia arriba al cambiar de página
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const goToDetail = (digimonName) => {
  router.push(`/home/digimon/${digimonName}`)
}

onMounted(() => {
  fetchDigimons()
})
</script>

<style scoped>
.digimon-card {
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
  cursor: pointer;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.digimon-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.digimon-image {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

/* Ajustes responsivos adicionales */
@media (max-width: 599px) {
  .digimon-card {
    font-size: 0.875rem;
  }
}

@media (min-width: 1920px) {
  .digimon-card {
    max-width: 100%;
  }
}
</style>
