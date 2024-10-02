<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer">
      <Menu  titulo="Menu principal" /> <!--Filho = Menu, comunica com ele com props-->
    </v-navigation-drawer>
    <v-app-bar>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-app-bar-title>Eventos</v-app-bar-title>
    </v-app-bar>

    <v-main>
      <v-container>
        <RouterView />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'

const drawer = ref(true)
</script>
