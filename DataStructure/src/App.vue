<template>
    <section v-if="indexId === 0">
        <div class="logo"><img class="logoPic" src="@/assets/icon/myLogo.svg" /></div>
        <div class="caption"><span>邻接矩阵</span></div>

        <ul class="options">
            <li @click="buildTable(1)">建立临界矩阵</li>
        </ul>
    </section>
    <section v-else-if="indexId === 1">
        <adjTab></adjTab>
    </section>
</template>

<script setup>
import { ref } from 'vue'
import adjTab from "@/components/adjTab.vue"

const indexId = ref(0)

const buildTable = (index) => {
    indexId.value = index
    // console.log(indexId)
}
</script>

<style>
/* body {
    background-color: rgb(128, 128, 255);
} */

.logo {
    width: 200px;
    height: 200px;
    background-color: rgba(255, 255, 255, 0.5);
    position: absolute;
    top: 30%;
    left: 50%;
    margin-left: -100px;
    margin-top: -100px;
    /* box-shadow: 0 0 20px rgba(0, 0, 0, 0.2); */
}

.logo img {
    width: 90%;
    height: 90%;
    padding-left: 35px;
    padding-top: 40px;
}

.caption {
    height: 60px;
    width: 400px;
    font-family: STZhongsong;
    /*font-weight: bold;*/
    font-size: 56px;
    font-weight: bold;
    letter-spacing: 5px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    color: white;
    text-align: center;
    position: absolute;
    top: 45%;
    left: 50%;
    margin-left: -200px;
}

.options {
    display: block;
    width: 600px;
    height: 300px;
    list-style-type: none;
    position: absolute;
    top: 55%;
    left: 50%;
    margin-left: -300px;
}

.options li {
    display: block;
    width: 300px;
    height: 75px;
    background-color: rgba(255, 255, 255, 0.4);
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    border-radius: 1000px;
    font-family: STZhongsong;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    color: white;
    font-size: 36px;
    margin: auto;
    margin-top: 50px;
    text-align: center;
    padding-top: 17px;
    transition: box-shadow 0.5s ease;
}

.options li:hover {
    box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
}
</style>