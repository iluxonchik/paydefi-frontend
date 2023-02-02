<script setup>
import {ref} from "vue";
const props = defineProps(["web3", "paymentRequestAddr", "paymentRequestAbi", "connectedAccountAddr"]);
import TokenAmountForm from "@/components/input/TokenAmountForm.vue";
</script>

<template>
<TokenAmountForm :web3="web3" :payment-request-addr="paymentRequestAddr" :payment-request-abi="paymentRequestAbi" :connected-account-addr="connectedAccountAddr">
</TokenAmountForm>
</template>
