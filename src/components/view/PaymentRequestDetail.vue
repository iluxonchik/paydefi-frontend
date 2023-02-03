<script setup>
import {computed, onMounted, ref} from "vue";

const props = defineProps(["paymentRequestId", "web3", "paymentRequestAddr", "paymentRequestAbi", "connectedAccountAddr"]);

const web3 = ref(props.web3);
const paymentRequestAddr = ref(props.paymentRequestAddr);
const paymentRequestAbi = ref(props.paymentRequestAbi);
const connectedAccountAddr = ref(props.connectedAccountAddr);
const paymentRequestId = ref(props.paymentRequestId);
const PaymentRequest = new web3.value.eth.Contract(paymentRequestAbi.value, paymentRequestAddr.value);
const isPaymentRequestStatic = computed(() => {
   return PaymentRequest.methods.isTokenAmountStatic(paymentRequestId.value).call().then(
       (result) => {
            return result;
       }
   )
});

function getStaticTokenAddressAndAmounts() {
    return PaymentRequest.methods.isTokenAmountStatic(paymentRequestId.value).call().then(
       (result) => {
            return result;
       }
   )
});
}


</script>
<template>
ID: <v-chip>
    {{ paymentRequestId }}
</v-chip>
    <div v-if="isPaymentRequestStatic">
        Payment Request Is Static
    </div>
</template>



