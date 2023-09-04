<template>
    <div class="input-container"></div>
    <div class="sort-container">
        <div>
            <el-table :data="tableData" id="dragTable" border style="width: 800px;">
                <el-table-column fixed prop="date" label="Date" width="150" />
                <el-table-column prop="name" label="Name" width="120" />
                <el-table-column prop="address" label="Address" width="600" />
                <el-table-column prop="zip" label="Zip" width="120" />
                <el-table-column fixed="right" label="Operations" width="120">
                    <template #default="scope">
                        <el-button link type="primary" size="small" @click.prevent="deleteRow(scope.$index)">
                            Remove
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-button class="mt-4" style="width: 100%" @click="onAddItem">Add Item</el-button>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import Sortable from 'sortablejs'
import { onMounted } from 'vue'

const tableData = [
    {
        date: '2016-05-03',
        name: 'Tom',
        address: 'No. 189, Grove St, Los Angeles',
    },
    {
        date: '2016-05-02',
        name: 'Cilly',
        address: 'No. 189, Grove St, Los Angeles',
    },
    {
        date: '2016-05-04',
        name: 'Linda',
        address: 'No. 189, Grove St, Los Angeles',
    },
    {
        date: '2016-05-01',
        name: 'John',
        address: 'No. 189, Grove St, Los Angeles',
    },
]

function setSort() {
    const el = document.querySelector('#dragTable table tbody')
    new Sortable(el, {
        sort: true,
        ghostClass: 'sortable-ghost',
        onEnd: (e) => {
            const targetRow = tableData.splice(e.oldIndex, 1)[0]
            tableData.splice(e.newIndex, 0, targetRow)
            console.log(tableData)
        },
    })
}
onMounted(() => {
    setSort()
})

const deleteRow = (index) => {
    tableData.value.splice(index, 1)
}

const onAddItem = () => {
    now.setDate(now.getDate() + 1)
    tableData.value.push({
        date: dayjs(now).format('YYYY-MM-DD'),
        name: 'Tom',
        address: 'No. 189, Grove St, Los Angeles',
        zip: 'CA 90036',
    })
}

</script>

<style>
.sort-container {
    margin: 30px;
    width: 50%;
}

</style>