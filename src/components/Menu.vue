<template>
  <v-list>
    <v-list-subheader>
      {{ titulo }}
    </v-list-subheader>
    <v-list-item prepend-icon="mdi-home" title="Home" to="/" />
    <v-list-item prepend-icon="mdi-calendar-plus" title="Cadastro" to="/Cadastro" />
  </v-list>
  <v-divider></v-divider>
</template>

<script setup>

const props = defineProps({
  titulo: String,
})
</script>
