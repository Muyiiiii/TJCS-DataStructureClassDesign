<template>
    <div v-if="!numCheck" class="input-container">
        <div class="inputNum-container">
            <p>请输入图中点的数量</p>
            <el-input-number v-model="num" :min="1" :max="7" @change="handleChange"/>
            <el-button type="primary" @click="numCheckChange(true)">确认</el-button>
        </div>
    </div>
    <div v-else>
        <h1>添加有向边</h1>
        <p>起始节点</p>
        <el-input-number v-model="a" :min="1" :max="num" controls-position="right" size="large"
                         @change="handleChange"/>
        <p>结束节点</p>
        <el-input-number v-model="b" :min="1" :max="num+100" controls-position="right" size="large"
                         @change="handleChange"/>
        <el-button type="primary" @click="addEdge()" size="large" style="margin-left: 30px">添加有向边</el-button>

        <div class="adjTab">
            <table>
                <tr v-for="(row, rowIndex) in g" :key="rowIndex">
                    <td v-for="(cell, columnIndex) in row" :key="columnIndex">
                        {{ cell }}
                    </td>
                </tr>
            </table>
        </div>
    </div>
</template>

<script setup>
import {ref} from 'vue'
import {ElMessage} from 'element-plus'

const num = ref(0)
const numCheck = ref(false) //num输入确认标志
const a = ref(0) //有向边的起始节点
const b = ref(0) //有向边的结束节点
let g = NaN
const handleChange = (value) => {
    console.log(value)
}

function numCheckChange(check) {
    g = Array.from(Array(num.value), () => new Array(num.value).fill(false))
    numCheck.value = check;
    console.log(g.length, g[0].length)
}

function addEdge() {
    if (a.value <= 0 || a.value > num.value || b.value <= 0 || b.value > num.value) {
        ElMessage.error('起始节点和结束节点的编号应该大于0且小于等于' + num.value)
    } else {
        ElMessage({message: '有向边添加成功.', type: 'success',})
        g[a.value - 1][b.value - 1] = true;
        console.log(g[a.value - 1][b.value - 1]);
    }
}

</script>

<style>
.input-container {
    /* height: 300px;
    width: 300px;
    background-color: black; */
}

.adjTab {
    margin-top: 5px;
}
</style>