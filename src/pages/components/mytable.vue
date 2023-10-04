<template >
    <DatatableV1 v-if="dataLoaded" :options="datatableV1" />
</template>



<script setup lang="ts">
import { reactive, onMounted, ref, computed } from 'vue'

import axios from 'axios'

const dataLoaded = ref(false)
const datatableV1Loaded = ref(false)

// interface res  {
//     name: string;
//     amount: string;
//     color?:string;
//     status?:boolean
// }

onMounted(() => {
    fetchJsonData()
})

let resdata
let datatableV1: object
const fetchJsonData = async (): Promise<void> => {
    await axios.get('https://mocki.io/v1/9919cda5-0d4d-46ae-bb96-d60cf283905b').then((response) => {
        console.log(response.data);
        resdata = response.data
        datatableV1 = {
            perPageSelect: [5, 10, 20, 25, 50, 100],
            perPage: 10,
            columns: [
                { select: 0, hidden: false },
                { select: 1, render: renderDate },
                { select: 2, render: renderAmount },
                { select: 3, render: renderButton, sortable: false },
            ],
            data: {
                headings: ['ID', 'Date', 'Amount', 'Actions'],
                data: resdata.data
            },
        }
        dataLoaded.value = true;
        console.log(datatableV1)
    })
}






const formatAmount = (amount: string): string => {
    return '$' + amount
}

function renderDate(data: any /*, cell: any, row: any */) {
    return `<span class="has-dark-text dark-inverted is-font-alt is-weight-600 rem-90">${data}</span>`
}

// Position
function renderAmount(data: any /*, cell: any, row: any */) {
    return `<span class="light-text">${data}</span>`
}

// Status
function renderStatus(data: any /*, cell: any, row: any */) {
    return `
          <div class="status is-${data}">
              <i aria-hidden="true" class="fas fa-circle"></i>
              <span class="is-capitalize">${data}</span>
          </div>
      `
}

// Button
function renderButton(data: any, cell: any, row: any) {
    return `<div class="has-text-right"><button class="button v-button is-dark-outlined" data-row="${row.dataIndex}">Manage</button></div>`
}

</script>