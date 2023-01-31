<script setup>
import Web3 from 'web3';
import {computed, watch, onMounted, ref} from "vue";

const emit = defineEmits(['walletConnected']);

const accounts = ref(null);
watch(accounts, async (newValue, oldValue) => {
    if (newValue !== null) {
        console.log("Connected to Wallet with: ", accounts);
        emit('walletConnected', {
            accounts: newValue,
            web3: new Web3(window.ethereum),
        });
    }
});

function connectToWeb3() {
    if (window.ethereum) {
       // if Metamask is installed (it creates a window.ethereum object)

        window.ethereum.request({
            method: 'eth_requestAccounts'
        }).then((connectedAccounts) => {
            console.log("Connected accounts: ", connectedAccounts)
            accounts.value = connectedAccounts;
        });
    }
}

onMounted(() => {
    console.log("Loaded");
    connectToWeb3();
});

</script>
<template>
    Requesting connection to your wallet...
    Refresh the page to retry.
</template>
