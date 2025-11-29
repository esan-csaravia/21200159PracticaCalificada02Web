<template>
  <div class="fullscreen flex flex-center bg-gradient">
    <q-card class="login-card q-pa-md shadow-10">
      <q-card-section class="text-center">
        <div class="digimon-container">
          <img src="../assets/digimon.png" alt="Digimon" class="digimon-image" />
        </div>
        <div class="text-h4 text-weight-bold text-primary q-mt-md">Digimon Login</div>
        <div class="text-subtitle2 text-grey-7">Ingresa tus credenciales</div>
      </q-card-section>

      <q-card-section>
        <q-form @submit.prevent="onSubmit" class="q-gutter-md">
          <q-input
            v-model="email"
            type="email"
            label="Email"
            outlined
            dense
            :rules="[
              (val) => !!val || 'El email es requerido',
              (val) => /.+@.+\..+/.test(val) || 'Email debe ser válido',
            ]"
          >
            <template v-slot:prepend>
              <q-icon name="email" />
            </template>
          </q-input>

          <q-input
            v-model="password"
            :type="isPwd ? 'password' : 'text'"
            label="Password"
            outlined
            dense
            :rules="[(val) => !!val || 'El password es requerido']"
          >
            <template v-slot:prepend>
              <q-icon name="lock" />
            </template>
            <template v-slot:append>
              <q-icon
                :name="isPwd ? 'visibility_off' : 'visibility'"
                class="cursor-pointer"
                @click="isPwd = !isPwd"
              />
            </template>
          </q-input>

          <div>
            <q-btn
              type="submit"
              label="Iniciar Sesión"
              color="primary"
              class="full-width"
              :loading="loading"
              size="md"
            />
          </div>
        </q-form>
      </q-card-section>
    </q-card>
  </div>
</template>

<script>
import { ref } from 'vue'
import { useQuasar } from 'quasar'
import { useRouter } from 'vue-router'
import axios from 'axios'

export default {
  name: 'LoginPage',
  setup() {
    const $q = useQuasar()
    const router = useRouter()
    const email = ref('')
    const password = ref('')
    const isPwd = ref(true)
    const loading = ref(false)

    const onSubmit = async () => {
      loading.value = true
      try {
        const response = await axios.post(
          'https://storedb-api.onrender.com/node-api/users/signin',
          {
            email: email.value,
            password: password.value,
          },
        )

        // Guardar el token o información del usuario
        if (response.data) {
          if (response.data.token) {
            localStorage.setItem('token', response.data.token)
          }
          if (response.data.user) {
            localStorage.setItem('user', JSON.stringify(response.data.user))
          }
        }

        $q.notify({
          type: 'positive',
          message: 'Login exitoso',
          position: 'top',
        })

        // Redirigir a la página principal después de un pequeño delay
        console.log('Redirecting to /home...')
        setTimeout(() => {
          router.push('/home').then(() => {
            console.log('Navigation successful')
          }).catch((err) => {
            console.error('Navigation error:', err)
          })
        }, 500)
      } catch (error) {
        $q.notify({
          type: 'negative',
          message: error.response?.data?.message || 'Error al iniciar sesión',
          position: 'top',
        })
      } finally {
        loading.value = false
      }
    }

    return {
      email,
      password,
      isPwd,
      loading,
      onSubmit,
    }
  },
}
</script>

<style scoped>
.bg-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.login-card {
  width: 100%;
  max-width: 400px;
  border-radius: 16px;
}

.digimon-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.digimon-image {
  width: 150px;
  height: 150px;
  object-fit: contain;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
}
</style>
