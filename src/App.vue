<script setup>
import { ref } from 'vue';


const theme = ref('light');

function toggleTheme() {
    theme.value = theme.value === 'light' ? 'dark' : 'light';
}

</script>

<template>
  <v-app :theme="theme">
      <v-app-bar
          :elevation="2"
          rounded
          title="PayDeFi - Send & Receive Money"
        >
            <v-spacer></v-spacer>
            <v-btn @click="toggleTheme">{{ theme == 'light' ? '🌙' : '☀️' }}</v-btn>
        </v-app-bar>

      <v-navigation-drawer
          expand-on-hover
          permanent
          rail
      >
          <v-list>
              <v-list-item
                  prepend-avatar="https://randomuser.me/api/portraits/women/85.jpg"
                  title="0x1235234523461234"
                  subtitle="Polygon Mainnet ✅"
              >
              </v-list-item>
              <v-divider></v-divider>
          </v-list>


      </v-navigation-drawer>
      <v-main></v-main>

      <v-bottom-navigation>
          <v-btn value="send">
            <v-icon icon="mdi-bank-transfer-out"></v-icon>
              Send
          </v-btn>

          <v-btn value="receive">
            <v-icon icon="mdi-bank-transfer-out"></v-icon>
              Receive
          </v-btn>
      </v-bottom-navigation>

  </v-app>
</template>
