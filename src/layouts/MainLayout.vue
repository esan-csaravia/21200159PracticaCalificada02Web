<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn flat dense round icon="menu" aria-label="Menu" @click="toggleLeftDrawer" />

        <q-toolbar-title> Digimon App </q-toolbar-title>

        <q-btn flat round dense icon="logout" @click="handleLogout">
          <q-tooltip>Cerrar Sesión</q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header> Menú Principal </q-item-label>

        <q-item clickable @click="goToHome">
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Inicio</q-item-label>
            <q-item-label caption>Galería de Digimones</q-item-label>
          </q-item-section>
        </q-item>

        <q-separator />

        <q-item clickable @click="handleLogout">
          <q-item-section avatar>
            <q-icon name="logout" color="negative" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Cerrar Sesión</q-item-label>
            <q-item-label caption>Volver al login</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { Notify, Dialog } from 'quasar'

const router = useRouter()
const leftDrawerOpen = ref(false)

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value
}

function goToHome() {
  router.push('/home')
  leftDrawerOpen.value = false
}

function handleLogout() {
  Dialog.create({
    title: 'Cerrar Sesión',
    message: '¿Estás seguro de que deseas cerrar sesión?',
    cancel: {
      label: 'Cancelar',
      flat: true,
    },
    ok: {
      label: 'Cerrar Sesión',
      color: 'negative',
    },
  }).onOk(() => {
    // Limpiar localStorage
    localStorage.removeItem('token')
    localStorage.removeItem('user')

    Notify.create({
      type: 'info',
      message: 'Sesión cerrada correctamente',
      position: 'top',
    })

    // Redirigir al login
    router.push('/')
  })
}
</script>
