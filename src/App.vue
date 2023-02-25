<script setup>
import {ref, computed, onMounted, reactive, onBeforeMount} from 'vue';
import Connect from "@/components/Connect.vue";
import Send from "@/components/Send.vue";
import Receive from "@/components/Receive.vue";
import PaymentRequestDetail from "@/components/view/PaymentRequestDetail.vue";
import PaymentTokenSelection from "@/components/PaymentTokenSelection.vue";
import Receipt from "@/components/Receipt.vue";

const ActiveComponent = {
    Connect: 0,
    Send: 1,
    Receive: 2,
    PaymentRequestDetail: 3,
    TokenAmountSend: 4,
    Receipt: 5,
}

const theme = ref('light');
const paymentRequestAddr = ref("0x2A8f490Be9a7CbbFA567727B81B968Ba8E1b6464");
const paymentRequestAbi = ref(null);
const web3 = ref(null);
const connectedAccountAddr = ref(null);
const connectedNetwork = ref(null);
const selectedActiveComponent = ref(0);
const selectedPaymentRequestIdForPayment = ref(null);
const receiptAddr = ref(null);
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

        getPaymentRequestContract().then(
            (paymentRequestContract) => {
                paymentRequestContract.methods.receipt().call(
                    {from: connectedAccountAddr.value}
                ).then(result => {
                    receiptAddr.value = result;
                    selectedActiveComponent.value = ActiveComponent.Receipt;
                    //selectedActiveComponent.value = ActiveComponent.Send;
                });
            }
        )
    });
}

function handlePaymentRequestIdSelected(data) {
    const selectedPaymentRequestId = data.selectedPaymentRequestId;
    selectedPaymentRequestIdForPayment.value = selectedPaymentRequestId;
    selectedActiveComponent.value = ActiveComponent.TokenAmountSend;
}

function handleCreatePaymentRequest(data) {
    activePaymentRequestDetail.paymentRequestId = data.createdPaymentRequestId;
    selectedActiveComponent.value = ActiveComponent.PaymentRequestDetail;
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

async function getPaymentRequestAbi() {
    const response = await fetch('/src/resources/payment_request_abi.json');
    return await response.json();
}

async function getPaymentRequestContract() {
    const paymentRequestAbi = await getPaymentRequestAbi();
    return new web3.value.eth.Contract(
        paymentRequestAbi.abi,
        paymentRequestAddr.value,
    )
}

onBeforeMount(async () => {
    // TODO: make async, through the use of a loading indicator
    const jsonResponse = await getPaymentRequestAbi()

  paymentRequestAbi.value = jsonResponse.abi;
  console.log("PR Abi: " + paymentRequestAbi.value);
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
                      <Receipt v-if="selectedActiveComponent === ActiveComponent.Receipt"
                               :web3="web3"
                               :connectedAccountAddr="connectedAccountAddr"
                               :receiptAddr="receiptAddr"
                               :receiptId="4"

                      />
                      <PaymentTokenSelection
                                v-if="selectedActiveComponent === ActiveComponent.TokenAmountSend"
                               :paymentRequestId="selectedPaymentRequestIdForPayment"
                               :web3="web3"
                               :paymentRequestAddr="paymentRequestAddr"
                               :paymentRequestAbi="paymentRequestAbi"
                               :connected-account-addr="connectedAccountAddr"
                      />
                      <Send
                            v-if="selectedActiveComponent === ActiveComponent.Send"
                            @paymentRequestIdSelected="handlePaymentRequestIdSelected"
                      />
                      <PaymentRequestDetail
                                v-if="selectedActiveComponent === ActiveComponent.PaymentRequestDetail"
                               :paymentRequestId="activePaymentRequestDetail.paymentRequestId"
                               :web3="web3"
                               :paymentRequestAddr="paymentRequestAddr"
                               :payment-request-abi="paymentRequestAbi"
                               :connected-account-addr="connectedAccountAddr"/>
                      <Receive
                                v-if="selectedActiveComponent === ActiveComponent.Receive"
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
            <v-icon icon="mdi-bank-transfer-in"></v-icon>
              Receive
          </v-btn>
      </v-bottom-navigation>

  </v-app>
</template>
