<template>
    <div class="common-layout">
        <el-container>
            <el-aside width="200px">
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
                                        <div v-if="rowIndex == columnIndex">
                                            <el-button type="info" :icon="Message" circle/>
                                        </div>
                                        <div v-else>
                                            <div v-if="cell">
                                                <el-button type="success" :icon="Check" circle
                                                           @click="changeEdgeByClick(rowIndex, columnIndex, 'success')"/>
                                            </div>
                                            <div v-else>
                                                <el-button type="danger" :icon="Delete" circle
                                                           @click="changeEdgeByClick(rowIndex, columnIndex, 'danger')"/>
                                            </div>
                                        </div>
                                    </td>
                                </tr>
                            </table>
                        </div>
                        <p>遍历的起始节点</p>
                        <el-input-number v-model="startNode" :min="1" :max="num" @change="handleChange"/>
                        <el-button type="success" round @click="bfs(matrix,startNode,onVisit)">开始BFS遍历
                        </el-button>
                    </div>
                </div>
                <div v-for="edge in graphData.links" style="background: aqua">
                    {{ edge.source }} {{ edge.target }}
                </div>
            </el-aside>
            <el-main>
                <div id="graph-container">
                    <svg id="svg"></svg>
                </div>
            </el-main>
        </el-container>
    </div>
</template>

<script setup>
import {ref, onMounted, watch} from 'vue' //VE是elmentui，ref是变量，vue是修改g用的
import {ElMessage} from 'element-plus' //发送提示信息
import {Check, Delete, Message} from '@element-plus/icons-vue'
import * as d3 from 'd3'

const num = ref(0);
const numCheck = ref(false);//num输入确认标志
const a = ref(0);//有向边的起始节点
const b = ref(0);//有向边的结束节点
//选择bfs和dfs开始的结点
const startNode = ref(0)
// 存储图的节点和边的信息
const matrix = ref([]);
const graphData = ref({
    nodes: [],
    links: []
});


const handleChange = (value) => {
    console.log(value);
}

function numCheckChange(check) {
    matrix.value = Array.from(Array(num.value), () => new Array(num.value).fill(false));
    numCheck.value = check;

    //这里的group是用来确定点的颜色的
    for (let i = 0; i < num.value; i++) {
        graphData.value.nodes.push({id: i, group: i});
    }

    console.log('matrix Info: ', matrix.value.length, matrix.value[0].length);
}

function addEdgeByInput() {
    if (a.value <= 0 || a.value > num.value || b.value <= 0 || b.value > num.value) {
        ElMessage.error('起始节点和结束节点的编号应该大于0且小于等于' + num.value);
    } else {
        ElMessage({message: '有向边添加成功.', type: 'success',});
        matrix.value[a.value - 1][b.value - 1] = true;
        graphData.value.links.push({source: a.value - 1, target: b.value - 1, value: 5});
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

        graphData.value.links = graphData.value.links.filter(edge => {
            return !(edge.source === rowIndex && edge.target === columnIndex);
        });
    } else {
        matrix.value[rowIndex][columnIndex] = true;
        matrix.value[columnIndex][rowIndex] = true;

        graphData.value.links.push({source: rowIndex, target: columnIndex, value: 100});

        console.log('source:', rowIndex, 'target:', columnIndex);
    }
}

watch(graphData.value, (newData, oldData) => {
    // 清除旧的图形
    d3.select("#svg").selectAll("*").remove();

    initGraph(newData)
});

onMounted(() => {
    initGraph(graphData.value)
});

const color = d3.scaleOrdinal(d3.schemeCategory10);
const width = 1000;
const height = 600;

function initGraph(data) {
    // Specify the color scale.

    // The force simulation mutates links and nodes, so create a copy
    // so that re-evaluating this cell produces the same result.
    const links = data.links.map(d => ({...d}));
    const nodes = data.nodes.map(d => ({...d}));

    // Create a simulation with several forces.
    const simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).strength(0.06))
        .force("charge", d3.forceManyBody().strength(-100))
        .force("center", d3.forceCenter(width / 2, height / 2))
        .on("tick", ticked);

    // Create the SVG container.
    const svg = d3.select("#svg")
        .attr("width", width)
        .attr("height", height)
        // .attr("viewBox", [0, 0, width, height])
        .attr("style", "max-width: 100%; height: auto;background-color:white");

    // Add a line for each link, and a circle for each node.
    const link = svg.append("g")
        .attr("stroke", "#999")
        .attr("stroke-opacity", 0.6)
        .selectAll()
        .data(links)
        .join("line")
        // .attr("stroke-width", d => Math.sqrt(d.value));
        .attr("stroke", "green")  // 将所有线的颜色更改为红色
        .attr("stroke-width", 15);

    const nodeGroup = svg.append("g")
        .selectAll(".node-group")
        .data(nodes)
        .join("g")
        .attr("class", "node-group")
        .call(d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended));

    nodeGroup.append("circle")
        .attr("r", 20)
        .attr("fill", d => color(d.group))
        .attr("id", d => "node-" + d.id);

    nodeGroup.append("text")
        .attr("dy", "0.35em")
        .attr("text-anchor", "middle")
        .text(d => d.id);

    // Set the position attributes of links and nodes each time the simulation ticks.
    function ticked() {
        link
            .attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y);

        nodeGroup
            .attr("transform", d => `translate(${d.x}, ${d.y})`);
    }

    // Reheat the simulation when drag starts, and fix the subject position.
    function dragstarted(event) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        event.subject.fx = event.subject.x;
        event.subject.fy = event.subject.y;
    }

    // Update the subject (dragged node) position during drag.
    function dragged(event) {
        event.subject.fx = event.x;
        event.subject.fy = event.y;
    }

    // Restore the target alpha so the simulation cools after dragging ends.
    // Unfix the subject position now that it’s no longer being dragged.
    function dragended(event) {
        if (!event.active) simulation.alphaTarget(0);
        event.subject.fx = null;
        event.subject.fy = null;
    }

}

function bfs(adjMatrix, startNode, onVisit) {
    let visited = new Array(adjMatrix.value.length).fill(false);
    let queue = [];

    queue.push(startNode);
    visited[startNode] = true;

    while (queue.length > 0) {
        let currentNode = queue.shift();
        onVisit(currentNode); // Callback to handle node visit

        for (let i = 0; i < adjMatrix.value[currentNode].length; i++) {
            if (adjMatrix[currentNode][i] && !visited[i]) {
                queue.push(i);
                visited[i] = true;
            }
        }
    }
}

function onVisit(nodeIndex) {
    d3.select(`#node-${nodeIndex}`).attr("fill", "aqua");
}

// // Use BFS and change color of nodes in D3 graph when they are visited
// bfs(matrix.value, 0, (nodeIndex) => {
//     d3.select(`#node-${nodeIndex}`).attr("fill", "red"); // Assuming nodes have IDs like node-0, node-1, etc.
// });

</script>

<style>
#input-container,
#graph-container {
    font-size: 16px;
    /* 或你想要的其他值 */
    vertical-align: top;
    margin: 30px;
}

.adjTab {
    margin-top: 5px;
}

.unselectable {
    user-select: none;
    -webkit-user-select: none; /* Safari */
    -moz-user-select: none; /* Firefox */
    -ms-user-select: none; /* IE/Edge */
}

</style>