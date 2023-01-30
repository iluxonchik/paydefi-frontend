<script setup>
import { ref } from 'vue';

const theme = ref('light');

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light';
}

</script>

<template>
  <v-app :theme="theme">
    <v-app-bar title="PayDeFi - Send & Receive Money">
      <v-spacer></v-spacer>
      <v-btn @click="toggleTheme">{{ theme == 'light' ? '🌙' : '☀️' }}</v-btn>
    </v-app-bar>

  </v-app>
</template>
