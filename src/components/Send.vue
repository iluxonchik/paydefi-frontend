<script setup>
import {computed, ref} from "vue";
const emit = defineEmits(["paymentRequestIdSelected"]);
const selectedPaymentRequestId = ref(null);
const nextBtnName = ref("nextBtn");
const isNextBtnDisabled = computed(() => {
    return selectedPaymentRequestId.value === null || selectedPaymentRequestId.value == "";
});
const rules = [
    (value) => {return parseInt(value) > 0 ? true : "Payment Request ID must be larger than 0"}
]

async function onSelectPaymentRequestId(e) {
    if (e.submitter.name === nextBtnName.value) {
        // emit event to parent
        emit("paymentRequestIdSelected",
            {
                selectedPaymentRequestId: parseInt(selectedPaymentRequestId.value),
            })
    }
}
</script>
<template>
    <VContainer>
        <VForm @submit.prevent="onSelectPaymentRequestId">
            <VRow>
                <VCol cols="12">
                    <VTextField
                        type="number"
                        v-model="selectedPaymentRequestId"
                        label="Payment Request ID"
                    />
                </VCol>
                <VCol cols="12">
                    <VBtn block type="submit" color="green" :name="nextBtnName" :disabled="isNextBtnDisabled">
                        Next
                    </VBtn>
                </VCol>
            </VRow>
        </VForm>
    </VContainer>
</template>
