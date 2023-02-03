<script setup>
import {computed, onMounted, ref} from "vue";

const props = defineProps(["web3", "paymentRequestAddr", "paymentRequestAbi", "connectedAccountAddr"]);

const web3 = ref(props.web3);
const paymentRequestAddr = ref(props.paymentRequestAddr);
const paymentRequestAbi = ref(props.paymentRequestAbi);
const connectedAccountAddr = ref(props.connectedAccountAddr);
const PaymentRequest = ref(null);

const numPaymentRequestsCreated = ref(0);

const addTokenBtnName = ref("addToken");
const createPaymentRequestBtnName = ref("createPaymentRequest");

const emit = defineEmits(['createPaymentRequest']);
const tokenAddr = ref("0x");
const tokenAmount = ref();
const tokenAmounts = ref([]);

const isCreateButtonDisabled = computed(
    () => {
       return tokenAmounts.value.length < 1;
    }
);

function refreshNumberOfCreatedPaymentRequests() {
    PaymentRequest.value = new web3.value.eth.Contract(paymentRequestAbi.value, paymentRequestAddr.value);
    PaymentRequest.value.methods.balanceOf(connectedAccountAddr.value).call({from: connectedAccountAddr.value}).then( result => numPaymentRequestsCreated.value = parseInt(result));
}

onMounted( () => {
   refreshNumberOfCreatedPaymentRequests();
});

function onAddTokenAmount(e) {
    // TODO: validate that token is a valid ERC-20 (in another part?)
    if (e.submitter.name === addTokenBtnName.value) {
        tokenAmounts.value.push({addr: tokenAddr.value, amount: tokenAmount.value});
        tokenAddr.value = "0x";
        tokenAmount.value = null;
    } else if (e.submitter.name === createPaymentRequestBtnName.value) {
        let tokensArg = [];
        tokenAmounts.value.forEach((value) => {
            tokensArg.push([value.addr, value.amount]);
        })
        PaymentRequest.value.methods.createWithStaticTokenAmount(tokensArg, '0x0000000000000000000000000000000000000000', '0x0000000000000000000000000000000000000000').send( {from: connectedAccountAddr.value}).then(result => refreshNumberOfCreatedPaymentRequests());
        emit("createPaymentRequest", tokenAmounts);
    }


}
</script>
<template>
        <v-form @submit.prevent="onAddTokenAmount">
        <v-container>
            <v-row v-for="token in tokenAmounts">
                <v-col cols="6">
                    {{ token.addr }}
                </v-col>

                <v-col cols="3">
                    {{ token.amount }}
                </v-col>
            </v-row>
            <v-row>
                <v-col cols="12" md="6">
                    <v-text-field
                        v-model.trim="tokenAddr"
                        label="Token Address"
                        hint="Address of The ERC-20 Token"
                        required
                    >
                    </v-text-field>

                </v-col>
                <v-col cols="12" md="3">
                    <v-text-field
                        v-model.trim="tokenAmount"
                        label="Token Amount"
                        hint="Amount of the ERC-20 Token"
                        type="number"
                        required
                    />
                </v-col>
            </v-row>
        </v-container>
            <v-container>
                <v-row>
                    <v-col cols="10" md="8"></v-col>
                    <v-col cols="1" align-self="end">
                        <v-btn type="submit" color="green" :name="addTokenBtnName">
                            Add
                        </v-btn>
                    </v-col>
                </v-row>

                <v-row>
                    <v-col md="9">
                        <v-btn block type="submit" color="green" :name=createPaymentRequestBtnName :disabled="isCreateButtonDisabled">
                            Create Request
                        </v-btn>
                    </v-col>
                </v-row>
                <p>Using Contract: {{ paymentRequestAddr }}</p>
                <p>You have created {{ numPaymentRequestsCreated }} Payment Requests</p>
            </v-container>
    </v-form>
</template>

