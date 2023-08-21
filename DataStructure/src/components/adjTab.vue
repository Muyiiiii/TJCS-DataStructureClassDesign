<template>
    <div v-if="!numCheck" class="input-container">
        <div class="inputNum-container">
            <p>请输入图中点的数量</p>
            <el-input-number v-model="num" :min="0" :max="7" @change="numChange"/>
            <el-button type="primary" @click="numCheckChange(true)">确认</el-button>
        </div>
    </div>
    <div v-else>
        <el-button type="primary" @click="numCheckChange(false)">确认</el-button>
    </div>
</template>

<script setup>
import {ref} from 'vue'

const num = ref(0)
const numCheck = ref(false) // num输入确认标志
let arr = NaN
const numChange = (value) => {
    console.log(value)

}

function numCheckChange(check) {
    arr = Array.from(Array(num.value), () => new Array(num.value))
    numCheck.value = check;
    console.log(arr.length)
}

</script>

<style>
.input-container {
    /* height: 300px;
    width: 300px;
    background-color: black; */
}
</style>