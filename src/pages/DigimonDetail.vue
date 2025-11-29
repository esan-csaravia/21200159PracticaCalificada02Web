<template>
  <q-page class="q-pa-md">
    <!-- Loading -->
    <div v-if="loading" class="flex flex-center q-pa-xl">
      <q-spinner-dots color="primary" size="50px" />
    </div>

    <!-- Detalle del Digimon -->
    <div v-else-if="digimon" class="row justify-center">
      <div class="col-12 col-md-10 col-lg-8">
        <!-- Botón Volver -->
        <q-btn
          flat
          icon="arrow_back"
          label="Volver"
          color="primary"
          @click="goBack"
          class="q-mb-md"
        />

        <q-card class="digimon-detail-card">
          <div class="row">
            <!-- Imagen -->
            <div class="col-12 col-md-5">
              <q-img
                :src="digimon.img"
                :alt="digimon.name"
                class="digimon-detail-image"
                fit="contain"
              >
                <template v-slot:loading>
                  <q-spinner-gears color="primary" />
                </template>
              </q-img>
            </div>

            <!-- Información -->
            <div class="col-12 col-md-7">
              <q-card-section>
                <div class="text-h3 text-h4-sm text-weight-bold text-primary q-mb-md">
                  {{ digimon.name }}
                </div>

                <q-separator class="q-my-md" />

                <div class="info-section">
                  <div class="info-item">
                    <q-icon name="stars" color="orange" size="sm" class="q-mr-sm" />
                    <span class="info-label">Nivel:</span>
                    <q-chip color="primary" text-color="white" dense>
                      {{ digimon.level }}
                    </q-chip>
                  </div>
                </div>

                <q-separator class="q-my-md" />

                <!-- Información adicional -->
                <div class="text-body1 q-mt-md">
                  <p class="text-grey-8">
                    <strong>{{ digimon.name }}</strong> es un Digimon de nivel
                    <strong>{{ digimon.level }}</strong
                    >. Los Digimon de este nivel poseen características únicas y habilidades
                    especiales que los hacen destacar en el mundo digital.
                  </p>
                </div>

                <!-- Acciones -->
                <div class="q-mt-lg">
                  <q-btn
                    color="primary"
                    label="Agregar a Favoritos"
                    icon="favorite"
                    outline
                    class="q-mr-sm q-mb-sm"
                  />
                  <q-btn color="secondary" label="Compartir" icon="share" outline class="q-mb-sm" />
                </div>
              </q-card-section>
            </div>
          </div>
        </q-card>
      </div>
    </div>

    <!-- Error -->
    <div v-else class="text-center q-pa-xl">
      <q-icon name="error_outline" size="64px" color="negative" />
      <div class="text-h6 q-mt-md">No se pudo cargar el Digimon</div>
      <q-btn color="primary" label="Volver" @click="goBack" class="q-mt-md" />
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import { Notify } from 'quasar'

const route = useRoute()
const router = useRouter()
const digimon = ref(null)
const loading = ref(false)

const fetchDigimonDetail = async () => {
  loading.value = true
  try {
    const digimonName = route.params.name
    const response = await axios.get(
      `https://digimon-api.vercel.app/api/digimon/name/${digimonName}`,
    )

    if (response.data && response.data.length > 0) {
      digimon.value = response.data[0]
    } else {
      throw new Error('Digimon no encontrado')
    }
  } catch (error) {
    console.error('Error fetching digimon detail:', error)
    Notify.create({
      type: 'negative',
      message: 'Error al cargar los detalles del Digimon',
      position: 'top',
    })
  } finally {
    loading.value = false
  }
}

const goBack = () => {
  router.push('/home')
}

onMounted(() => {
  fetchDigimonDetail()
})
</script>

<style scoped>
.digimon-detail-card {
  overflow: hidden;
}

.digimon-detail-image {
  height: 400px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.info-section {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.info-item {
  display: flex;
  align-items: center;
  font-size: 1.1rem;
}

.info-label {
  font-weight: 600;
  margin-right: 0.5rem;
  color: #5a5a5a;
}

@media (max-width: 959px) {
  .digimon-detail-image {
    height: 300px;
  }
}

@media (max-width: 599px) {
  .digimon-detail-image {
    height: 250px;
  }

  .info-item {
    font-size: 1rem;
  }
}
</style>
