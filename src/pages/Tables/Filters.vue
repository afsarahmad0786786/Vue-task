<template>
    <div style="width: 12%;" class="my-5">
        <select title="Select your Filter" class="select" aria-label="Filter" @input="updateValue">
            <option selected disabled value="default">Search By Filter</option>
            <option value="Name">Name</option>
            <option value="Date">Date</option>
            <option value="Amount">Amount</option>
            <option value="Address">Address</option>
        </select>
    </div>
</template>

<script setup lang="ts">

const props = defineProps({
    modelValue: {
        type: String,
        required: true,
    },
})

const emit = defineEmits(['update:modelValue']);

const updateValue = (e: Event) => {
    emit('update:modelValue', (e.target as HTMLInputElement).value)
};

</script>