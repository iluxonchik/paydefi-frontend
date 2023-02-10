<script setup>
import {ref} from "vue";
const props = defineProps(["web3", "paymentRequestAddr", "paymentRequestAbi", "connectedAccountAddr"]);
const emit = defineEmits(["createPaymentRequest"])
import TokenAmountForm from "@/components/input/TokenAmountForm.vue";
</script>

<template>
<TokenAmountForm :web3="web3"
                 :payment-request-addr="paymentRequestAddr"
                 :payment-request-abi="paymentRequestAbi"
                 :connected-account-addr="connectedAccountAddr"
                 @createPaymentRequest="$emit('createPaymentRequest', $event)"
>
</TokenAmountForm>
</template>
