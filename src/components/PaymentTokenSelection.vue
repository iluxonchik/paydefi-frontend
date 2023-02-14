<script setup>

import {computed, onMounted, reactive, ref} from "vue";

const props = defineProps([
    "paymentRequestId",
    "web3",
    "paymentRequestAddr",
    "paymentRequestAbi",
    "connectedAccountAddr"
]);

const web3 = ref(props.web3);
const paymentRequestAddr = ref(props.paymentRequestAddr);
const paymentRequestId = ref(props.paymentRequestId);
const paymentRequestAbi = ref(props.paymentRequestAbi);
const connectedAccountAddr = ref(props.connectedAccountAddr);
const isPaymentRequestStatic = ref(false);
const PaymentRequest = new web3.value.eth.Contract(
    paymentRequestAbi.value,
    paymentRequestAddr.value,
);

const selectedToken = ref(null);
const payBtnName = ref("payBtn");
const isPayBtnDisabled = computed(() => {
    return false;
});

const acceptedTokenAmounts = ref([]);

const tokenAmountsSelect = computed( () => {
    return acceptedTokenAmounts.value.map(
        (elem) => {
            return {
                title: elem.token + " | " + elem.tokenAmount,
                token: elem.token
            }
    });
});



async function setIsPaymentRequestStatic() {
    isPaymentRequestStatic.value = await PaymentRequest.methods.isTokenAmountStatic(paymentRequestId.value).call();
}

async function getStaticTokenAddressAmounts() {
    acceptedTokenAmounts.value = await PaymentRequest.methods.getStaticTokenAmountInfos(paymentRequestId.value).call();

}


onMounted( () => {
    setIsPaymentRequestStatic();
    getStaticTokenAddressAmounts();

})

</script>

<template>
    <VContainer>
        <VForm>
            <VRow>
                <VCol cols="12">
                    <VSelect
                        v-model="selectedToken"
                        :items="tokenAmountsSelect"
                        item-title="title"
                        item-value="token"
                        :hint="`Actual Name For Token ${selectedToken}`"
                        label="Select Payment Token"
                        persistent-hint
                        >
                    </VSelect>
                </VCol>
                <VCol cols="12">
                    <VBtn block type="submit" color="green" :name="payBtnName" :disabled="isPayBtnDisabled">
                        Pay
                    </VBtn>
                </VCol>
            </VRow>
        </VForm>
    </VContainer>

</template>
