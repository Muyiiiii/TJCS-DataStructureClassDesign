<template>
    <div v-if="showId === 0" class="input-container">
        <el-row>
            <el-col :span="12">
                <div class="show-container">
                    <!-- <div class="demo-collapse"> -->
                    <p style="font-size: 15px;">(鼠标放到行上1的零件上，可以通过拖拽来决定安装步骤，顺序是由上往下)</p>
                    <el-collapse v-model="activeNames" @change="handleChange">
                        <section v-for="item in itemList" :key="item.id">
                            <el-collapse-item :title="item.name" :name="item.id">
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
                                    <div style="margin:1%;">
                                        <el-input v-model="newPath" placeholder="输入要加入到当前产品的零件" style="width: 68%;" />
                                        <el-button class="mt-4" style="width: 28%;background-color:aquamarine"
                                            @click="addPath(item.id)">Add Item</el-button>
                                    </div>
                                </div>
                            </el-collapse-item>
                        </section>
                    </el-collapse>
                    <!-- </div> -->
                </div>
            </el-col>
            <el-col :span="12">
                <div class="card">
                    <div class="chose-container1">
                        <el-input v-model="newItem" placeholder="请输入要添加的新产品的名称" size="large"
                            style="width:70%; margin-bottom:5px;" />
                        <el-button type="success" size="large" @click="addItem(newItem)" :style="{ float: 'right' }"
                            style="display: block;">添加物品</el-button>
                    </div>
                    <div class="chose-container2">
                        <el-select-v2 v-model="itemOnAssemble" :options="itemOptions" placeholder="请选择要维修的产品" size="large"
                            style="width: 70%;" />
                        <el-button type="primary" size="large" @click="changeShowId(1)"
                            :style="{ float: 'right' }">开始检修</el-button>
                    </div>
                </div>
            </el-col>
        </el-row>
    </div>
    <div v-else-if="showId === 1" class="assemble-container">
        <el-tag type="success" class="mx-1" effect="dark" round style="margin-left:10%;">
            <p>所有零件</p>
        </el-tag>
        <div class="tags1">
            <div v-for="path in itemList[itemOnAssemble].list" style="display: inline-flex;">
                <el-row>
                    <el-col :span="4">
                        <el-tag v-if="path.value === partChosen" class="ml-2" type="danger">{{ path.value }}</el-tag>
                        <el-tag v-else>{{ path.value }}</el-tag>
                    </el-col>
                </el-row>
            </div>

        </div>
        <div class="fixChose">
            <el-tag type="danger" class="mx-1" effect="dark" round>
                <p>请选择要维修的零件</p>
            </el-tag>
            <el-autocomplete v-model="partChosen" :fetch-suggestions="querySearch" placeholder="请选择要维修的零件"
                @select="handleSelect">
                <template #default="{ item }">
                    <span>{{ item.value }}</span>
                </template>
            </el-autocomplete>
            <el-button type="primary" @click="() => { showInstall = showDismantle = true }">开始维修</el-button>
        </div>

        <el-button type="warning" round style="margin-left:10%;display: block;" size="large"
            @click="() => showDismantle = !showDismantle">
            <p>拆解路径显示开关</p>
        </el-button>
        <div class="tags2" v-show="showDismantle">
            <div v-for="path in itemList[itemOnAssemble].list.slice(partChosenIdx).reverse()">
                <el-tag v-if="path.value === partChosen" class="ml-2" type="danger">{{ path.value }}</el-tag>
                <el-tag v-else>{{ path.value }}</el-tag>
            </div>
        </div>
        <el-button type="warning" round style="margin-left:10%;margin-top: 5%;" size="large"
            @click="() => showInstall = !showInstall">
            <p>安装路径显示开关</p>
        </el-button>
        <div class="tags2" v-show="showInstall">
            <div v-for="path in itemList[itemOnAssemble].list.slice(partChosenIdx)">
                <el-tag v-if="path.value === partChosen" class="ml-2" type="danger">{{ path.value }}</el-tag>
                <el-tag v-else>{{ path.value }}</el-tag>
            </div>
        </div>
        <el-button type="info" round style="margin-left:80%;margin-top: 5%;" size="large" @click="changeShowId(0)">
            <p>返回</p>
        </el-button>
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

const showInstall = ref(false)
const showDismantle = ref(false)

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
        partChosen.value = ''
        partChosenIdx.value = ''
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
        name: '第一个默认物体',
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
    width: 100%;
    margin: 20px;
}

div {
    border-radius: 10px;
}

.chose-container1 {
    margin: 20px;
}

.chose-container2 {
    margin: 60px 20px;
}

.assemble-container {
    width: 100%;
    height: 100%;
    margin: 2% 0%;
    /* background: linear-gradient(315deg, #42d392 25%, #647eff); */
    background: linear-gradient(315deg, #42d392 25%, #647eff);
}

.tags1 {
    /* background: linear-gradient(315deg, #42d392 25%, #647eff); */
    background: #FAFCFF;
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    max-height: 20%;
    border: #AF40FF;
    width: 80%;
    margin: 0.5% 10%;
    /* Assuming each row is 20px */
    overflow-y: auto;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);

    /* Show vertical scrollbar when content exceeds max-height */
}

.tags2 {
    /* background: linear-gradient(315deg, #42d392 25%, #647eff); */
    background: #FAFCFF;
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    max-height: 20%;
    border: #AF40FF;
    width: 80%;
    margin: 1% 10% 2% 10%;
    /* Assuming each row is 20px */
    overflow-y: auto;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);

    /* Show vertical scrollbar when content exceeds max-height */
}

.fixChose {
    margin: 2% 10%;
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