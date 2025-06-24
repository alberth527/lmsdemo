<template>
  <div id="q-app">
    <q-layout view="hHh lpR fFf">
      <q-header elevated>
        <q-toolbar>
          <q-toolbar-title>Reward Demo</q-toolbar-title>
        </q-toolbar>
      </q-header>
      <q-page-container>
        <q-page class="q-pa-md">
          <div v-if="!loggedIn">
            <q-form @submit.prevent="login">
              <q-input v-model="loginForm.username" label="Username" />
              <q-input v-model="loginForm.password" type="password" label="Password" />
              <q-btn label="Login" type="submit" color="primary" class="q-mt-md" />
            </q-form>
          </div>
          <div v-else>
            <q-tabs v-model="tab" dense class="text-grey-8">
              <q-tab name="points" label="Points" />
              <q-tab name="products" label="Products" />
              <q-tab name="history" label="History" />
            </q-tabs>
            <q-separator class="q-my-md" />
            <q-tab-panels v-model="tab" animated>
              <q-tab-panel name="points">
                <div class="text-h6">Current Points: {{ user.points }}</div>
                <q-btn label="Logout" color="primary" @click="logout" class="q-mt-md" />
              </q-tab-panel>
              <q-tab-panel name="products">
                <q-card v-for="p in products" :key="p.id" class="q-mb-md">
                  <q-img :src="p.image" :alt="p.name" height="150px" />
                  <q-card-section>
                    <div class="text-h6">{{ p.name }}</div>
                    <div>{{ p.description }}</div>
                    <div class="text-subtitle2 q-mt-sm">Requires {{ p.points }} pts</div>
                  </q-card-section>
                  <q-card-actions>
                    <q-btn label="Redeem" color="primary" @click="openRedeem(p)" />
                  </q-card-actions>
                </q-card>
              </q-tab-panel>
              <q-tab-panel name="history">
                <q-list bordered>
                  <q-item v-for="h in user.history" :key="h.time">
                    <q-item-section>{{ h.name }}</q-item-section>
                    <q-item-section side>{{ h.points }} pts</q-item-section>
                    <q-item-section side>{{ formatDate(h.time) }}</q-item-section>
                  </q-item>
                </q-list>
              </q-tab-panel>
            </q-tab-panels>
          </div>
        </q-page>
      </q-page-container>
      <q-dialog v-model="redeemDialog">
        <q-card>
          <q-card-section>
            <div class="text-h6">Redeem {{ selectedProduct.name }} for {{ selectedProduct.points }} pts?</div>
          </q-card-section>
          <q-card-actions align="right">
            <q-btn flat label="Cancel" v-close-popup />
            <q-btn label="Confirm" color="primary" @click="redeem" />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </q-layout>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const loginForm = ref({ username: '', password: '' })
const tab = ref('points')
const redeemDialog = ref(false)
const selectedProduct = ref({})
const user = ref(JSON.parse(localStorage.getItem('reward-user')) || null)

const products = ref([
  { id: 1, name: 'Coupon 100', points: 100, image: 'https://via.placeholder.com/300', description: 'Discount coupon' },
  { id: 2, name: 'Gift 1', points: 300, image: 'https://via.placeholder.com/300', description: 'Gift item' }
])

function saveUser() {
  localStorage.setItem('reward-user', JSON.stringify(user.value))
}

const fakeUser = { username: 'demo', password: '1234', points: 500, history: [] }

function login() {
  if (loginForm.value.username === fakeUser.username && loginForm.value.password === fakeUser.password) {
    user.value = { ...fakeUser }
    saveUser()
  } else {
    alert('Invalid credentials')
  }
}

function logout() {
  user.value = null
  localStorage.removeItem('reward-user')
}

function openRedeem(p) {
  selectedProduct.value = p
  redeemDialog.value = true
}

function redeem() {
  const p = selectedProduct.value
  if (user.value.points >= p.points) {
    user.value.points -= p.points
    user.value.history.push({ name: p.name, points: p.points, time: Date.now() })
    saveUser()
    redeemDialog.value = false
    alert('Redeemed successfully!')
  } else {
    alert('Not enough points')
  }
}

function formatDate(t) {
  return new Date(t).toLocaleString()
}

const loggedIn = computed(() => user.value !== null)
</script>

<style>
body, #q-app {
  min-height: 100vh;
}
</style>
