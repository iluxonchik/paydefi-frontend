<script setup>
import {ref} from "vue";

const emit = defineEmits(['tokenAdd']);
const tokenAddr = ref("0x");
const tokenAmount = ref();
const tokenAmounts = ref([]);

function onAddTokenAmount(e) {
    // TODO: validate that token is a valid ERC-20 (in another part?)
    tokenAmounts.value.push({addr: tokenAddr.value, amount: tokenAmount.value});
    tokenAddr.value = "0x";
    tokenAmount.value = null;
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
                        <v-btn type="submit" color="green">
                            Add
                        </v-btn>
                    </v-col>

                </v-row>
            </v-container>
    </v-form>
</template>

