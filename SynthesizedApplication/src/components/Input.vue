<template>
    <div v-if="showId === 0" class="input-container">
        <el-row>
            <el-col :span="12">
                <div class="show-container">
                    <!-- <div class="demo-collapse"> -->
                    <el-collapse v-model="activeNames" @change="handleChange">
                        <section v-for="item in itemList" :key="item.id">
                            <el-collapse-item :title="item.name + '(鼠标放到ID列和行上试试 可以拖拽行和列)'" :name="item.id">
                                <div style="width:100%;">
                                    <table class="tb">
                                        <thead>
                                            <draggable v-model="item.headers" animation="200" tag="tr"
                                                :item-key="(key) => key">
                                                <template #item="{ element: header }">
                                                    <th class="move">
                                                        {{ header }}
                                                    </th>
                                                </template>
                                            </draggable>
                                        </thead>
                                        <draggable :list="item.list" handle=".move" animation="300"
                                            @start="onStart(item.id)" @end="onEnd(item.id)" tag="tbody" item-key="name">
                                            <template #item="{ element }">
                                                <tr>
                                                    <td class="move" v-for="header in item.headers" :key="header">
                                                        {{ element[header] }}
                                                    </td>
                                                </tr>
                                            </template>
                                        </draggable>
                                    </table>
                                    <el-input v-model="newPath" placeholder="Please input" style="width: 70%;" />
                                    <el-button class="mt-4" style="width: 30%;background-color: aqua"
                                        @click="addPath(item.id)">Add Item</el-button>
                                </div>
                            </el-collapse-item>
                        </section>
                    </el-collapse>
                    <!-- </div> -->
                </div>
            </el-col>
            <el-col :span="12">
                <div class="card">
                    <div class="chose-container">
                        <el-input v-model="newItem" placeholder="Please input" />
                        <el-button type="success" size="large" @click="addItem(newItem)">添加物品</el-button>
                    </div>
                    <div class="chose-container">
                        <el-select-v2 v-model="itemOnAssemble" :options="itemOptions" placeholder="Please select"
                            size="large" />
                        <el-button type="primary" size="large" @click="changeShowId(1)">开始检测</el-button>
                    </div>
                </div>
            </el-col>
        </el-row>
    </div>
    <div v-else-if="showId === 1" class="assemble-container">
        <div class="tags">
            <div v-for="path in itemList[itemOnAssemble].list">
                <el-tag v-if="path.value === partChosen" class="ml-2" type="danger">{{ path.value }}</el-tag>
                <el-tag v-else>{{ path.value }}</el-tag>
            </div>
        </div>
        <p>请选择要维修的零件</p>
        <el-autocomplete v-model="partChosen" :fetch-suggestions="querySearch" placeholder="请选择" @select="handleSelect">
            <template #default="{ item }">
                <span>{{ item.value }}</span>
            </template>
        </el-autocomplete>
        <el-button type="primary">开始维修</el-button>
        <div class="tags">
            <p>拆解路径</p>
            <div v-for="path in itemList[itemOnAssemble].list.slice(partChosenIdx).reverse()">
                <el-tag v-if="path.value === partChosen" class="ml-2" type="danger">{{ path.value }}</el-tag>
                <el-tag v-else>{{ path.value }}</el-tag>
            </div>
        </div>
        <div class="tags">
            <p>安装路径</p>
            <div v-for="path in itemList[itemOnAssemble].list.slice(partChosenIdx)">
                <el-tag v-if="path.value === partChosen" class="ml-2" type="danger">{{ path.value }}</el-tag>
                <el-tag v-else>{{ path.value }}</el-tag>
            </div>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { ref, reactive, computed } from 'vue'
import draggable from "vuedraggable";
import { ElMessage } from 'element-plus'

const showId = ref(0)
const activeNames = ref(0)
const generalId = ref(1)
const newPath = ref('')
const newItem = ref('')
const itemOnAssemble = ref()
// value:选项的值,也就是选中该选项后绑定的 model 值
// label: 选项的标签, 也就是显示在页面上的文本
const itemOptions = computed(() => {
    return itemList.map(item => {
        return {
            value: item.id,
            label: item.name
        }
    })
})

const partChosen = ref('')
const partChosenIdx = ref(-1)

const querySearch = (queryString, cb) => {

    // 过滤出包含queryString的选项
    const includeResults = itemList[itemOnAssemble.value].list.filter(i => i.value.includes(queryString))

    // 过滤出剩余不包含的选项
    const excludeResults = itemList[itemOnAssemble.value].list.filter(i => !i.value.includes(queryString))

    const results = queryString
        ? [...includeResults, ...excludeResults]
        : itemList[itemOnAssemble.value].list
    cb(results)
    console.log(itemList[0].list)
    console.log(queryString)
    console.log(results)
}

const handleSelect = (item) => {
    partChosen.value = item.value
    partChosenIdx.value = itemList[itemOnAssemble.value].list.findIndex(d => d.value === item.value)
    console.log(item)
}

const changeShowId = (id) => {
    if (id === 1 && itemList[itemOnAssemble.value].list.length === 0) {
        ElMessage({ showClose: true, message: '要维修的产品的安装路径为空，请先填写安装路径', type: 'error', })
        activeNames.value = itemOnAssemble.value
        showId.value = 0
    } else {
        showId.value = id
    }
}

const handleChange = (val: string[]) => {
    // console.log(val)
    // console.log(itemOptions)
    // console.log(itemOnAssemble)
    console.log(activeNames)
}

/*
draggable 对CSS样式没有什么要求万物皆可拖拽
:list="state.list"         //需要绑定的数组
animation="300"            //动画效果
@start="onStart"           //拖拽开始的事件
@end="onEnd"               //拖拽结束的事件
*/
const itemList = reactive([
    {
        id: 0,
        name: '第一个物体',
        //列的名称
        headers: ["value"],
        //需要拖拽的数据，拖拽后数据的顺序也会变化
        list: [
            { value: "11111111" },
            { value: "22222222" },
            { value: "33333333" },
        ],
        pathCnt: 3,
    },
]);

//拖拽开始的事件
const onStart = (id) => {
    console.log("开始拖拽");
};

//拖拽结束的事件
const onEnd = (id) => {
    console.log("结束拖拽");
    console.log(itemList[id].list)
};

const addPath = (id) => {
    if (newPath.value === '') {
        ElMessage({ showClose: true, message: '添加的产品的零件名称不能为空！', type: 'error', })
    } else {
        itemList[id].list.push({ value: newPath.value })
        itemList[id].pathCnt++
        newPath.value = ''
    }
};

const addItem = (item) => {
    if (item === '') {
        ElMessage({ showClose: true, message: '添加的产品名称不能为空！', type: 'error', })
    } else {
        itemList.push({
            id: generalId.value++,
            name: item,
            //列的名称
            headers: ["value"],
            //需要拖拽的数据，拖拽后数据的顺序也会变化
            list: [],
            pathCnt: 3,
        });

        newItem.value = '';
    }
};
</script>

<style>
@import '@/assets/css/input.css';

.show-container {
    background-color: cornflowerblue;
    width: 100%;
    margin: 20px;
}

div {
    border-radius: 10px;
}

.chose-container {
    margin: 30px;
}

.assemble-container {
    width: 100%;
    height: 100%;
    margin: 0%;
    background: linear-gradient(315deg, #42d392 25%, #647eff);
}

.tags {
    display: flex;
}

.el-tag {
    font-size: 20px;
    height: 30px;
    padding: 10px;
    margin: 5px;

    /* display: flex;
    align-items: center;
    justify-content: center; */
}
</style>