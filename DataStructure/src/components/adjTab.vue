<template>
    <div v-if="!numCheck" class="input-container">
        <div class="inputNum-container">
            <p>请输入图中点的数量</p>
            <el-input-number v-model="num" :min="0" :max="7" @change="numChange" />
            <el-button type="primary">Primary</el-button>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const num = ref(0)
const numCheck = ref(false) // num输入确认标志

const numChange = (value) => {
    console.log(value)
    console.log(arr)
}

const arr = ref(Array.from(Array(num), () => new Array(num)))

</script>

<style>
.input-container {
    /* height: 300px;
    width: 300px;
    background-color: black; */
}
</style>