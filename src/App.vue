<script setup>
import { ref, computed } from 'vue';
import Connect from "@/components/Connect.vue";
import Send from "@/components/Send.vue";
import Receive from "@/components/Receive.vue";

const ActiveComponent = {
    Connect: 0,
    Send: 1,
    Receive: 2,
}

const theme = ref('light');
const connectedAccountAddr = ref(null);
const connectedNetwork = ref(null);
const selectedActiveComponent = ref(0);

const accountName = computed(() => {
    return connectedAccountAddr.value === null ? "No Account" : connectedAccountAddr.value;
});

const networkName = computed( () => {
    return connectedNetwork.value === null ? "Disconnected ❌" : connectedNetwork.value + " ✅";
});

function toggleTheme() {
    theme.value = theme.value === 'light' ? 'dark' : 'light';
}

function handleWalletConnected(data) {
    connectedAccountAddr.value = data.accounts[0];
    data.web3.eth.net.getNetworkType().then((name) => {
        name = name.toLowerCase();
        connectedNetwork.value = name.charAt(0).toUpperCase() + name.slice(1);
    });
    selectedActiveComponent.value = ActiveComponent.Send;

}

function selectSendComponent() {
    if (connectedAccountAddr !== null) {
        selectedActiveComponent.value = ActiveComponent.Send;
    }
}

function selectReceiveComponent() {
    if (connectedAccountAddr !== null) {
        selectedActiveComponent.value = ActiveComponent.Receive;
    }
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
                  :title="accountName"
                  :subtitle="networkName"
              >
              </v-list-item>
              <v-divider></v-divider>
          </v-list>

      </v-navigation-drawer>
      <v-main>
          <v-container>
              <v-row>
                  <v-col cols="12">
                      <Connect v-if="selectedActiveComponent === ActiveComponent.Connect" @wallet-connected="handleWalletConnected"/>
                      <Send v-if="selectedActiveComponent === ActiveComponent.Send" />
                      <Receive v-if="selectedActiveComponent === ActiveComponent.Receive" />
                  </v-col>
              </v-row>
          </v-container>
      </v-main>

      <v-bottom-navigation>
          <v-btn value="send" @click="selectSendComponent">
            <v-icon icon="mdi-bank-transfer-out"></v-icon>
              Send
          </v-btn>

          <v-btn value="receive" @click="selectReceiveComponent">
            <v-icon icon="mdi-bank-transfer-out"></v-icon>
              Receive
          </v-btn>
      </v-bottom-navigation>

  </v-app>
</template>
