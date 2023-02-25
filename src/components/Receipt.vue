<script setup>
    import {onBeforeMount} from "vue";

    const props = defineProps(
        [
            "web3",
            "receiptId",
            "receiptAddr",
            "connectedAccountAddr"
        ]);

    async function loadReceiptContract() {
        const response = await fetch('src/resources/receipt.json');
        const jsonResponse = await response.json();
        return new props.web3.eth.Contract(
            jsonResponse,
            props.receiptAddr,
        );
    }
    onBeforeMount(async () => {
        const receiptContract = await loadReceiptContract();

        receiptContract.methods.getReceiptData(
            props.receiptId,
        ).call({from: props.connectedAccountAddr}).then(
            (result) => {
                console.log("Obtained receipt data: ", result);
            }
        )
    })
    /*
        Receipt data:
        address paymentRequest;
        uint256 paymentRequestId;
        address token;
        uint256 tokenAmount;
        address payer;
        address payee;
        address beneficiary;
     */
</script>

<template>

</template>


