<script setup>

import {computed, onBeforeMount, onMounted, reactive, ref} from "vue";
import * as events from "events";

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
const PaymentRequest = computed( () => {
    if (paymentRequestAbi.value !== null) {
       return new web3.value.eth.Contract(
            props.paymentRequestAbi,
            props.paymentRequestAddr
       );
    } else {
        return null;
    }


});

const selectedToken = ref(null);
const payBtnName = ref("payBtn");
const acceptedTokenAmounts = ref([]);
const acceptedTokenToTokenAmount = computed( () => {
        // https://stackoverflow.com/questions/11508463/javascript-set-object-key-by-variable
        return acceptedTokenAmounts.value.reduce((tokenAmountDict, tokenElem) => Object.assign(tokenAmountDict, {[tokenElem.token]: tokenElem.tokenAmount}), {});
});

const isPayBtnDisabled = computed(() => {
    return false;
});


const tokenAmountsSelect = computed( () => {
    return acceptedTokenAmounts.value.map(
        (elem) => {
            return {
                title: elem.token + " | " + elem.tokenAmount,
                token: elem.token
            }
    });
});



function setIsPaymentRequestStatic() {
    if (PaymentRequest.value !== null) {
        console.log("not null lol");
        PaymentRequest.value.methods.isTokenAmountStatic(paymentRequestId.value).call().then(
            (resValue) => {
                isPaymentRequestStatic.value = resValue;
            }
        );
    }
}

function getStaticTokenAddressAmounts() {
    if (PaymentRequest.value !== null) {
        PaymentRequest.value.methods.getStaticTokenAmountInfos(paymentRequestId.value).call().then(
            (resValue) => {
                acceptedTokenAmounts.value = resValue;
            }
        );
    }
}

async function loadErc20ContractAbiForAddr(contractAddr) {
    const response = await fetch('/src/resources/erc20_abi.json');
    const jsonResponse = await response.json();

    return new web3.value.eth.Contract(
        jsonResponse,
        contractAddr
    );

}

async function onSelectPaymentToken(e) {
    if (e.submitter.name === payBtnName.value) {
        const selectedTokenAddr = selectedToken.value;
        const tokenAmount = acceptedTokenToTokenAmount.value[selectedTokenAddr];
        const erc20Contract = await loadErc20ContractAbiForAddr(selectedTokenAddr);

        erc20Contract.events.Approval(
            {filter: {address: connectedAccountAddr.value, spender: paymentRequestAddr.value }}
        ).on("data", function(event) {
            console.log("Approval event received data: " + JSON.stringify(event));
        });

        // TODO: increaseAllowance() is non-standard. Adapt use of approve()
        erc20Contract.methods.approve(
            paymentRequestAddr.value,
            tokenAmount
        ).send(
            {from: connectedAccountAddr.value}
        ).then((result) => {
            // TODO: perform error checking
            console.log(result);
            PaymentRequest.value.events.PaymentRequestPaid(
                {filter: {paymentRequestId: paymentRequestId.value, token: selectedTokenAddr, payer: connectedAccountAddr}}
            ).on("data", function (event){
                // TODO: emit event to parent and display payment receipt
                console.log("Payment Request Paid" + JSON.stringify(event));
            })

            PaymentRequest.value.methods.pay(props.paymentRequestId, selectedTokenAddr).send(
                {from: connectedAccountAddr.value}
            ).then((result) => {
                console.log("Payment Request payment result: " + JSON.stringify(result));
            })

        })


    }
}

onBeforeMount( () => {
    setIsPaymentRequestStatic();
    getStaticTokenAddressAmounts();

})

</script>

<template>
    <VContainer>
        <VForm @submit.prevent="onSelectPaymentToken">
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
