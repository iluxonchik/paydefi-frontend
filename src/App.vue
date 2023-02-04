<script setup>
import {ref, computed, onMounted, reactive} from 'vue';
import Connect from "@/components/Connect.vue";
import Send from "@/components/Send.vue";
import Receive from "@/components/Receive.vue";
import PaymentRequestDetail from "@/components/view/PaymentRequestDetail.vue";

const ActiveComponent = {
    Connect: 0,
    Send: 1,
    Receive: 2,
    PayementRequestDetail: 3,
}

const theme = ref('light');
const paymentRequestAddr = ref("0x2A8f490Be9a7CbbFA567727B81B968Ba8E1b6464");
const paymentRequestAbi = ref(null);
const web3 = ref(null);
const connectedAccountAddr = ref(null);
const connectedNetwork = ref(null);
const selectedActiveComponent = ref(0);
const activePaymentRequestDetail = reactive({
    paymentRequestId: null,
})

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
    web3.value = data.web3;
    connectedAccountAddr.value = data.accounts[0];
    data.web3.eth.net.getNetworkType().then((name) => {
        name = name.toLowerCase();
        connectedNetwork.value = name.charAt(0).toUpperCase() + name.slice(1);
    });
    selectedActiveComponent.value = ActiveComponent.Send;
}

function handleCreatePaymentRequest(data) {
    activePaymentRequestDetail.paymentRequestId = data.createdPaymentRequestId;
    selectedActiveComponent.value = ActiveComponent.PayementRequestDetail;
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

onMounted(() => {
  fetch('/src/resources/payment_request_abi.json')
      .then( (response) => response.json())
      .then((json) => {
            paymentRequestAbi.value = json.abi;
      });
});

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
                      <PaymentRequestDetail v-if="selectedActiveComponent === ActiveComponent.PayementRequestDetail"
                               :paymentRequestId="activePaymentRequestDetail.paymentRequestId"
                               :web3="web3"
                               :paymentRequestAddr="paymentRequestAddr"
                               :payment-request-abi="paymentRequestAbi"
                               :connected-account-addr="connectedAccountAddr"/>
                      <Receive v-if="selectedActiveComponent === ActiveComponent.Receive"
                               @createPaymentRequest="handleCreatePaymentRequest"
                               :web3="web3"
                               :paymentRequestAddr="paymentRequestAddr"
                               :payment-request-abi="paymentRequestAbi"
                               :connected-account-addr="connectedAccountAddr"
                      />
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
