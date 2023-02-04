<script setup>
import {computed, onMounted, ref} from "vue";

const props = defineProps(["paymentRequestId", "web3", "paymentRequestAddr", "paymentRequestAbi", "connectedAccountAddr"]);

const web3 = ref(props.web3);
const paymentRequestAddr = ref(props.paymentRequestAddr);
const paymentRequestAbi = ref(props.paymentRequestAbi);
const connectedAccountAddr = ref(props.connectedAccountAddr);
const paymentRequestId = ref(props.paymentRequestId);
const PaymentRequest = new web3.value.eth.Contract(paymentRequestAbi.value, paymentRequestAddr.value);
const isPaymentRequestStatic = ref(false);
const staticTokenAmounts = ref([]);


function setIfPaymentRequestIsStatic() {
   PaymentRequest.methods.isTokenAmountStatic(paymentRequestId.value).call().then(
       (result) => {
            isPaymentRequestStatic.value = result;
       }
   )
}

onMounted( () => {
    setIfPaymentRequestIsStatic();
    getStaticTokenAddressAndAmounts();
});

function getStaticTokenAddressAndAmounts() {
    return PaymentRequest.methods.getStaticTokenAmountInfos(paymentRequestId.value).call().then(
       (result) => {
           result.forEach((element) => staticTokenAmounts.value.push({token: element.token, tokenAmount: element.tokenAmount}));
       });
}

</script>
<template>
ID: <v-chip>
    {{ paymentRequestId }}
</v-chip>
    <div v-if="isPaymentRequestStatic">
        <v-list>
            <v-list-subheader>Accepted Static Token Amounts</v-list-subheader>
            <v-divider></v-divider>
            <v-list-item
                v-for="(item, i) in staticTokenAmounts"
                :key="i"
                :value="item.token"
                border="3"
                variant="elevated"
                rounded="true"
            >
                <v-list-item-title v-text="item.token"></v-list-item-title>
                <v-list-item-title v-text="item.tokenAmount"></v-list-item-title>
            </v-list-item>
        </v-list>
    </div>
</template>



