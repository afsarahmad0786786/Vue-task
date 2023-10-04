<template>
    <input :value="modelValue" @input="updateValue" class="input my-3" :placeholder="`Search by ${props.searchField}`" />
</template>

<script setup lang="ts">

const props = defineProps({
    modelValue: {
        type: String,
        required: true
    },
    searchField: {
        type: String,
        required: true,
        default: 'Search'
    }
})

const emit = defineEmits(['update:modelValue']);

const updateValue = (e: Event) => {
    emit('update:modelValue', (e.target as HTMLInputElement).value)
};

</script>