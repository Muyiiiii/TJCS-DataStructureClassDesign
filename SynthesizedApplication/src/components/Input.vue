<template>
    <div class="input-container">
        <div class="demo-collapse">
            <el-collapse v-model="activeNames" @change="handleChange">
                <el-collapse-item title="Consistency" name="1">
                    <div class="title">鼠标放到ID列和行上试试 可以拖拽行和列</div>
                    <div style="width:50%;">
                        <table class="tb">
                            <thead>
                                <draggable v-model="state.headers" animation="200" tag="tr" :item-key="(key) => key">
                                    <template #item="{ element: header }">
                                        <th class="move">
                                            {{ header }}
                                        </th>
                                    </template>
                                </draggable>
                            </thead>
                            <draggable :list="state.list" handle=".move" animation="300" @start="onStart" @end="onEnd"
                                       tag="tbody" item-key="name">
                                <template #item="{ element }">
                                    <tr>
                                        <td class="move" v-for="(header, index) in state.headers" :key="header">
                                            {{ element[header] }}
                                        </td>
                                    </tr>
                                </template>
                            </draggable>
                        </table>
                        <el-input v-model="newPath" placeholder="Please input" />
                        <el-button class="mt-4" style="width: 100%;background-color: aqua" @click="addPath">Add Item
                        </el-button>
                    </div>
                </el-collapse-item>
                <el-collapse-item title="Feedback" name="2">
                    <p>2222222222222222222222222222222222222222222222</p>
                </el-collapse-item>
            </el-collapse>
        </div>
    </div>
</template>

<script lang="ts" setup>
import {ref, reactive} from 'vue'
import draggable from "vuedraggable";

const activeNames = ref(['1'])
const handleChange = (val: string[]) => {
    console.log(val)
}

/*
draggable 对CSS样式没有什么要求万物皆可拖拽
:list="state.list"         //需要绑定的数组
animation="300"            //动画效果
@start="onStart"           //拖拽开始的事件
@end="onEnd"               //拖拽结束的事件
*/
const state = reactive({
    //列的名称
    headers: ["name"],
    //需要拖拽的数据，拖拽后数据的顺序也会变化
    list: [
        {name: "11111111"},
        {name: "22222222"},
        {name: "33333333"},
    ],
    cnt: 3,
});

const newPath=ref('')

//拖拽开始的事件
const onStart = () => {
    console.log("开始拖拽");
};

//拖拽结束的事件
const onEnd = () => {
    console.log("结束拖拽");
    console.log(state.list)
};

const addPath = () => {
    state.list.push({name:newPath.value})
    newPath.value=''

};
</script>

<style>
.input-container{
    background-color: cornflowerblue;
    width: 60%;
    margin: 20px;
}

.title {
    padding: 3px;
    font-size: 13px;
}

.itxst {
    width: 600px;
}

.move {
    cursor: move;
}

table.tb {
    color: #333;
    border: solid 1px #999;
    font-size: 13px;
    border-collapse: collapse;
    min-width: 500px;
    user-select: none;
}

table.tb th {
    background: rgb(168 173 217);
    border-width: 1px;
    padding: 8px;
    border-style: solid;
    border-color: #999;
    text-align: left;
}

table.tb th:nth-of-type(1) {
    text-align: center;
}

table.tb td {
    background: #d6c8c8;
    border-width: 1px;
    padding: 8px;
    border-style: solid;
    border-color: #999;
}

table.tb td:nth-of-type(1) {
    text-align: center;
}
</style>