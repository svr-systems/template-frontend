<template>
  <v-row align="center" justify="center" class="my-2">
    <v-icon :color="!isDark ? 'orange' : 'grey'">mdi-white-balance-sunny</v-icon>
    <v-switch
      v-model="darkMode"
      hide-details
      inset
      density="compact"
      class="mx-2"
      color="primary"
    />
    <v-icon :color="isDark ? 'blue-grey' : 'grey'">mdi-weather-night</v-icon>
  </v-row>
</template>

<script setup>
// Importaciones de librerías
import { ref, watch, computed } from 'vue'
import { useTheme } from 'vuetify'

// Importaciones internas
import { useStore } from '@/store'

const theme = useTheme()
const store = useStore()

// Computados y estado local
const isDark = computed(() => store.conf.theme_dark)
const darkMode = ref(isDark.value)

// Watchers
watch(darkMode, (val) => {
  store.themeDarkAction()
  theme.global.name.value = val ? 'dark' : 'light'
})

watch(
  isDark,
  (val) => {
    darkMode.value = val
  },
  { immediate: true }
)
</script>
