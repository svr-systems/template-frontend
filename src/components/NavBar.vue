<template>
  <div>
    <v-app-bar density="compact" :elevation="2">
      <v-app-bar-nav-icon v-if="isMobile" @click.stop="toggleDrawer" />
      <v-toolbar-title>{{ APP_NAME }}</v-toolbar-title>
      <v-spacer />
      <v-btn icon variant="text">
        <v-icon>mdi-bell-outline</v-icon>
      </v-btn>
      <v-btn icon variant="text" @click="profileDialog = true">
        <v-icon>mdi-account-circle</v-icon>
      </v-btn>
    </v-app-bar>

    <NavDrawer v-model="drawer" :is-mobile="isMobile" />
    <NavProfile v-model="profileDialog" />
  </div>
</template>

<script setup>
// Importaciones de librerías
import { ref, computed, watch } from 'vue'
import { useDisplay } from 'vuetify'

// Importaciones internas del proyecto
import { APP_NAME } from '@/utils/config'

// Componentes
import NavDrawer from './NavDrawer.vue'
import NavProfile from './NavProfile.vue'

// Estado y computados
const display = useDisplay()
const isMobile = computed(() => display.smAndDown.value)
const drawer = ref(!isMobile.value)
const profileDialog = ref(false)

// Funciones
const toggleDrawer = () => {
  drawer.value = !drawer.value
}

// Reacción a cambios de tamaño de pantalla
watch(isMobile, (val) => {
  drawer.value = !val
})
</script>
