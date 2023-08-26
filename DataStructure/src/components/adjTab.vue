<template>
    <div class="common-layout">
        <el-container>
            <el-aside width="400px">
                <div id="input-container">
                    <div v-if="!numCheck">
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
                        <el-input-number v-model="b" :min="1" :max="num" controls-position="right" size="large"
                                         @change="handleChange"/>
                        <el-button type="primary" @click="addEdgeByInput()" size="large" style="margin-left: 30px">添加有向边
                        </el-button>

                        <div class="adjTab">
                            <table>
                                <tr v-for="(row, rowIndex) in matrix" :key="rowIndex">
                                    <td v-for="(cell, columnIndex) in row" :key="columnIndex">
                                        <div v-if="cell">
                                            <el-button type="success" :icon="Check" circle
                                                       @click="changeEdgeByClick(rowIndex, columnIndex, 'success')"/>
                                        </div>
                                        <div v-else>
                                            <el-button type="danger" :icon="Delete" circle
                                                       @click="changeEdgeByClick(rowIndex, columnIndex, 'danger')"/>
                                        </div>
                                    </td>
                                </tr>
                            </table>
                        </div>
                    </div>
                </div>
                <div v-for="edge in edges" style="background: aqua">
                    {{ edge.source }} {{ edge.target }}
                </div>
            </el-aside>
            <el-main>
                <div id="graph-container">
                    <svg id="svg" style="background-color: brown;"></svg>
                </div>
            </el-main>
        </el-container>
    </div>


</template>

<script setup>
import {ref, onMounted} from 'vue' //VE是elmentui，ref是变量，vue是修改g用的
import {ElMessage} from 'element-plus' //发送提示信息
import * as d3 from 'd3'

const num = ref(0);
const numCheck = ref(false);//num输入确认标志
const a = ref(0);//有向边的起始节点
const b = ref(0);//有向边的结束节点
// 存储图的节点和边的信息
const matrix = ref([]);
const nodes = ref([]);
const edges = ref([]);


const handleChange = (value) => {
    console.log(value);
}

function numCheckChange(check) {
    matrix.value = Array.from(Array(num.value), () => new Array(num.value).fill(false));
    numCheck.value = check;

    for (let i = 0; i < num.value; i++) {
        nodes.value.push({id: i})
    }

    console.log(matrix.value.length, matrix.value[0].length);
}

function addEdgeByInput() {
    if (a.value <= 0 || a.value > num.value || b.value <= 0 || b.value > num.value) {
        ElMessage.error('起始节点和结束节点的编号应该大于0且小于等于' + num.value);
    } else {
        ElMessage({message: '有向边添加成功.', type: 'success',});
        matrix.value[a.value - 1][b.value - 1] = true;
        edges.value.push({source: a.value - 1, target: b.value - 1});
        console.log(matrix.value[a.value - 1][b.value - 1]);
    }
}

// 切换边的连接状态
function changeEdgeByClick(rowIndex, columnIndex, type) {
    console.log(`Button at row ${rowIndex} and column ${columnIndex} of type ${type} was clicked.`);
    // 这里你可以根据 rowIndex、columnIndex 和 type 执行更复杂的逻辑
    a.value = rowIndex + 1;
    b.value = columnIndex + 1;
    if (type === 'success') {
        matrix.value[rowIndex][columnIndex] = false;

        edges.value = edges.value.filter(edge => {
            return !(edge.source === rowIndex && edge.target === columnIndex);
        });
    } else {
        matrix.value[rowIndex][columnIndex] = true;

        edges.value.push({source: rowIndex, target: columnIndex});
    }
}

//开始画图
onMounted(() => {
    const svg = d3.select('#svg');
    let width = 800;
    let height = 600;
    svg.attr('width', width)
        .attr('height', height);

    //我们创建一个D3力导向模拟(forceSimulation)。这个模拟会计算每个节点的位置，并按照我们定义的物理规则（力）进行更新。
    const simulation = d3.forceSimulation(nodes.value)
        .force("link", d3.forceLink(edges.value).distance(100))
        .force("charge", d3.forceManyBody().strength(-200))
        .force("center", d3.forceCenter(width / 2, height / 2));

    const edgesGraph = svg.append("g")
        .selectAll("line")
        .data(edges.value)
        .enter().append("line")
        .attr("stroke", "black");

    const nodesGraph = svg.append("g")
        .selectAll("circle")
        .data(nodes.value)
        .enter().append("circle")
        .attr("r", 5)
        .attr("fill", "blue");

    simulation.on("tick", () => {
        edgesGraph.attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y);

        nodesGraph.attr("cx", d => d.x)
            .attr("cy", d => d.y);
    });

});
</script>

<style>
#input-container, #graph-container {
    font-size: 16px; /* 或你想要的其他值 */
    vertical-align: top;
    margin: 30px;
}

.adjTab {
    margin-top: 5px;
}
</style>