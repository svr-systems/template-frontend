<template>
  <v-row dense justify="center">
    <v-col cols="12" md="4" class="mt-11">
      <v-card elevation="24" :disabled="ldg" :loading="ldg">
        <v-card-text class="text-center">
          <v-row>
            <Logo />
            <v-col cols="12">
              <h2 class="font-weight-light">
                {{ !success ? 'Confirmar cuenta' : 'Cuenta confirmada' }}
              </h2>
            </v-col>
            <v-col v-if="!item" cols="12">
              <p v-if="ldg">Validando información</p>
              <div v-else>
                <p>
                  <v-icon size="x-large">mdi-alert-circle</v-icon>
                </p>
                <p>La cuenta ya está confirmada y/o la acción no es procesable</p>
              </div>
            </v-col>
            <v-col v-else cols="12">
              <v-row>
                <v-col cols="12">
                  <v-row>
                    <v-col v-if="!success" cols="12">
                      <v-form v-if="item" ref="itemForm">
                        <v-row dense>
                          <v-col cols="12">
                            <b>E-mail</b>
                            <br />
                            {{ item.email }}
                          </v-col>
                          <v-col cols="12">
                            <InpPassword
                              label="Contraseña"
                              v-model="item.password"
                              :rules="rules.password_rqd"
                              no-prepend-icon
                            />
                          </v-col>
                          <v-col cols="12">
                            <div class="text-right">
                              <v-btn
                                block
                                size="small"
                                color="success"
                                @click.prevent="handleAction"
                                :loading="ldg"
                              >
                                Confirmar Cuenta y contraseña
                                <template v-slot:append>
                                  <v-icon size="small">mdi-account-check</v-icon>
                                </template>
                              </v-btn>
                            </div>
                          </v-col>
                        </v-row>
                      </v-form>
                    </v-col>
                    <v-col v-else cols="12">
                      <p>
                        <v-icon size="x-large">mdi-check-circle</v-icon>
                      </p>
                      <p v-if="item.role_id != 3">
                        <small> Serás redireccionado a tu cuenta en breve </small>
                        <br />
                        <v-progress-circular size="16" indeterminate />
                      </p>
                      <p v-else>
                        Podrás iniciar sesión en el
                        <b>Módulo de Atención</b> indicado por el Hospital donde radica tu cuenta
                      </p>
                    </v-col>
                  </v-row>
                </v-col>
              </v-row>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script setup>
import { ref, onMounted, inject } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useStore } from '@/store'
import { URL_API } from '@/utils/config'
import { getHdrs, getErr, getRsp } from '@/utils/http'
import { getRules } from '@/utils/validators'
import { getObj } from '@/utils/helpers'
import axios from 'axios'
import BtnTheme from '@/components/BtnTheme.vue'
import Logo from '@/components/Logo.vue'
import InpPassword from '@/components/InpPassword.vue'

const route = useRoute()
const router = useRouter()
const store = useStore()
const alert = inject('alert')

const id = route.params.id
const ldg = ref(true)
const item = ref(null)
const rules = getRules()
const success = ref(false)
const itemForm = ref(null)

// Métodos
const getItem = async () => {
  try {
    const response = await axios.get(`${URL_API}/public/user/account_confirm/${id}`, getHdrs())
    const rsp = getRsp(response)
    item.value = { ...rsp.data.item, password: null }
  } catch (err) {
    getErr(err)
  } finally {
    ldg.value = false
  }
}

const handleAction = async () => {
  const { valid } = await itemForm.value.validate()

  if (valid) {
    const obj = getObj(item.value, true)
    ldg.value = true

    try {
      const response = await axios.post(
        `${URL_API}/public/user/account_confirm/${id}`,
        obj,
        getHdrs()
      )
      success.value = true
      setTimeout(async () => {
        try {
          const loginResponse = await axios.post(`${URL_API}/auth/login`, obj, getHdrs())
          const loginRsp = getRsp(loginResponse)
          store.loginAction(loginRsp.data.auth)
          router.push({ name: 'home' })
        } catch (err) {
          alert?.show('error', getErr(err))
        }
      }, 5000)
    } catch (err) {
      alert?.show('error', getErr(err))
    } finally {
      ldg.value = false
    }
  }
}

onMounted(() => {
  setTimeout(() => {
    getItem()
  }, 2500)
})
</script>