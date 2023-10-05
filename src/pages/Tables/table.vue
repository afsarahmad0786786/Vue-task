<script setup lang="ts">
import { reactive, onMounted, ref, computed } from 'vue';
import { usePagination } from "vue-composable";
import axios from 'axios';

/*
Components Imports
*/
import Search from './Search.vue';
import Filter from './Filters.vue';

const data: any = reactive({
    tableData: [],
    mobiledata: []
});

const searchValue = ref('');
const searchField = ref('');
const sizeOfData = ref('');
const sortColumn = ref('name');
const sortorder = ref('asc');
const loadMoreValue = ref(5);
const selectedValue = ref(5);
const isMobileDevice = ref(false);
const dataLoaded = ref(false);

const {
    currentPage,
    lastPage,
    next,
    prev,
    offset,
    pageSize,
} = usePagination({
    currentPage: 0,
    pageSize: computed(() => selectedValue.value),
    total: computed(() => data.tableData.length),
});

interface tableData {
    name: string;
    data: Date;
    amount: string;
    address: string

};

onMounted(async () => {
    await fetchJsonData();
    loadMore();
});

let timer = setInterval(() => {
    isMobile();
}, 100)

onBeforeUnmount(() => {
    clearInterval(timer)
})

const fetchJsonData = async (): Promise<void> => {
    await axios
        .get('https://dev-u4ad9h2ebgvuhwn.api.raw-labs.com/2')
        .then((response) => {
            data.tableData = response.data
            dataLoaded.value = true
            sizeOfData.value = data.tableData.length
        });
};

const formatAmount = (amount: string): string => {
    return '$' + amount
};

const deleteData = (name: string) => {
    let index: number = data.tableData.findIndex((e: tableData) => e.name === name);
    data.tableData.splice(index, 1)
};

const editData = (name: string) => {
    let index: number = data.tableData.findIndex((e: tableData) => e.name === name);
    data.tableData.splice(index, 1)
};

const results: any = computed(() => {
    if (searchValue.value) {
        console.log(searchField.value)
        let datav1;
        let searchData = data.tableData.filter((el: any) => {
            datav1 = el
            if (searchField.value === 'Name') {
                return datav1.name.toLowerCase().includes(searchValue.value.toLowerCase())
            } else if (searchField.value === 'Date') {
                return datav1.date.toLowerCase().includes(searchValue.value.toLowerCase())
            } else if (searchField.value === 'Amount') {
                return datav1.amount.toLowerCase().includes(searchValue.value.toLowerCase())
            } else if (searchField.value === 'Address') {
                return datav1.address.toLowerCase().includes(searchValue.value.toLowerCase())
            }
        });
        return searchData.slice(offset.value, offset.value + pageSize.value);
    }
    if (sortorder.value) {
        let sortedData = data.tableData.sort((a: any, b: any): number => {
            if (sortColumn.value === 'amount') {
                if (parseInt(a[sortColumn.value]) < parseInt(b[sortColumn.value])) {
                    return sortorder.value === "asc" ? -1 : 1
                } else if (parseInt(a[sortColumn.value]) > parseInt(b[sortColumn.value])) {
                    return sortorder.value === "asc" ? 1 : -1
                } else {
                    return 0
                }
            }
            if (sortColumn.value === 'date') {
                if (new Date(a[sortColumn.value]) < new Date(b[sortColumn.value])) {
                    return sortorder.value === "asc" ? -1 : 1
                } else if (new Date(a[sortColumn.value]) > new Date(b[sortColumn.value])) {
                    return sortorder.value === "asc" ? 1 : -1
                } else {
                    return 0
                }
            }
            if (a[sortColumn.value] < b[sortColumn.value]) {
                return sortorder.value === "asc" ? -1 : 1
            } else if (a[sortColumn.value] > b[sortColumn.value]) {
                return sortorder.value === "asc" ? 1 : -1
            } else {
                return 0
            }
        });
        return sortedData.slice(offset.value, offset.value + pageSize.value);
    }

});

const sortingByField = (sortingField: string): void => {
    sortorder.value = sortorder.value === 'asc' ? 'desc' : 'asc'
    switch (sortingField) {
        case 'name':
            sortColumn.value = "name"
            break;
        case 'address':
            sortColumn.value = "address"
            break;
        case 'amount':
            sortColumn.value = "amount"
            break;
        case 'date':
            sortColumn.value = "date"
            break;
    }
}

const isMobile = () => {
    if (screen.width <= 760) {
        isMobileDevice.value = true
    } else {
        isMobileDevice.value = false
    }
}

const loadMore = (): void => {
    let newMobileData = []
    console.log('load more' + loadMoreValue.value)
    let loopValue = loadMoreValue.value
    for (let i = 0; i < loopValue; i++) {
        newMobileData.push(data.tableData[i])
    }
    loadMoreValue.value += 5;
    data.mobiledata = newMobileData
}

const getSizeValue = (e: Event) => {
    let pagesize = (e.target as HTMLInputElement).value
    selectedValue.value = +pagesize
};
</script>

<template>
    <div v-if="isMobileDevice && dataLoaded">
        <!-- <Filter v-model="searchField" />
        <Search v-model="searchValue" :searchField="searchField" /> -->
        <div class="card has-text-centered is-dark">
            <h5 class="card-title p-2 mx-6">Employee Details</h5>
        </div>

        <div v-for="res in data.mobiledata" class="card border-success mb-3" style="max-width: 45rem;">
            <div class="card-body text-success">

                <h5 class="card-title text-dark">Name - {{ res.name }}</h5>
                <h5 class="card-title">Date - {{ res.date }}</h5>

                <h5 class="card-title">Address - {{ res.address }}</h5>
                <h5 class="card-title">Amount - ${{ res.amount }}</h5>
                <div class="box has-text-centered">
                    <VButton icon="fas fa-trash" @click="deleteData(res.name)" class="button" color="primary">
                        Delete
                    </VButton>
                </div>
            </div>
        </div>
        <div>
            <div class="box has-text-centered">
                <button :disabled="data.tableData.length <= loadMoreValue" class="button is-dark is-rounded mx-6"
                    @click="loadMore()">Load More ...</button>
            </div>
        </div>
    </div>


    <div v-if="dataLoaded && !isMobileDevice && results.length" class="container is-fluid" style="float: left">
        <span>
            <Filter v-model="searchField" />
            <Search v-model="searchValue" :searchField="searchField" />
        </span>
        <div style="width: 8%;" class="my-5">
            <select class="select" aria-label="Filter" @change="getSizeValue">
                <option selected value="5">5</option>
                <option value="10">10</option>
                <option value="15">15</option>
                <option value="20">20</option>
                <option value="25">25</option>
            </select>
        </div>
        <table class="table is-hoverable is-fullwidth">
            <thead>
                <tr>
                    <th scope="col">Name&nbsp; <button @click="sortingByField('name')" style="font-size:10px"> <i
                                class="fa fa-sort"></i></button></th>
                    <th scope="col">Date &nbsp;<button @click="sortingByField('date')" style="font-size:10px"> <i
                                class="fa fa-sort"></i></button></th>
                    <th scope="col">Amount&nbsp;<button @click="sortingByField('amount')" style="font-size:10px"> <i
                                class="fa fa-sort"></i></button></th>
                    <th scope="col">Address&nbsp;<button @click="sortingByField('address')" style="font-size:10px"> <i
                                class="fa fa-sort"></i></button></th>
                    <th scope="col">Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for=" res  in  results ">
                    <template v-if="res">
                        <td>{{ res?.name || '---' }}</td>

                        <td>{{ res?.date || '---' }}</td>

                        <td>{{ formatAmount(res?.amount || 0) }}</td>

                        <td>{{ res?.address || '---' }}</td>

                        <td>
                            <VButton icon="fas fa-trash" v-tooltip.warning="`You want to delete ${res.name} data`"
                                @click="deleteData(res.name)" class="button" color="primary">Delete</VButton>
                            <VButton icon="feather:edit-2" v-tooltip.warning="`You want to edit ${res.name} data`"
                                @click="editData(res.name)" class="button mx-2" color="primary">Edit</VButton>
                        </td>
                    </template>

                </tr>
            </tbody>
        </table>
        <div class="pagination is-rounded">
            <span class="is-rounded ">page {{ currentPage }} of {{ lastPage }} showing from {{ sizeOfData }}</span>
            <span>
                <button class="pagination-previous mx-1" @click="prev">prev</button>
                <button :disabled="currentPage >= lastPage" class="pagination-next " @click="next">next</button>
            </span>
        </div>
    </div>
</template>
