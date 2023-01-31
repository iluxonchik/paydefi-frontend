<script setup>
import {ref} from "vue";

const tokenAddr = ref("0x");
const tokenAmount = ref(null);

</script>

<template>
    <v-form>
        <v-container>
            <v-row>
                <v-col cols="8">
                    <v-text-field
                        v-model="tokenAddr"
                        label="Token Address"
                        hint="Address of the ERC-20 token"
                        required
                    >
                    </v-text-field>

                </v-col>
                <v-col cols="3">
                    <v-text-field
                        v-model="tokenAmount"
                        label="Token Amount"
                        hint="Amount of the ERC-20 token"
                        type="number"
                        required
                    />

                </v-col>
            </v-row>
        </v-container>
    </v-form>
</template>
