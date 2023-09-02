<template>
    <div class="common-layout">
        <el-container>
            <el-aside width="35%">
                <div class="input-container">
                    <div v-if="!numCheck">
                        <div class="inputNum-container">
                            <p>请输入图中点的数量</p>
                            <el-input-number v-model="num" :min="1" :max="7" @change="handleChange" />
                            <el-button type="primary" @click="numCheckChange(true)">确认</el-button>
                        </div>
                    </div>
                    <div v-else>
                        <el-row>
                            <el-col :span="16">
                                <h1>添加无向边</h1>
                                <div class="input-data">
                                    <div class="input-content">
                                        <p>起始节点</p>
                                    </div>
                                    <el-input-number v-model="a" :min="0" :max="num - 1" controls-position="right"
                                        size="large" @change="handleChange" />
                                </div>
                                <div class="input-data">
                                    <div class="input-content">
                                        <p>结束节点</p>
                                    </div>
                                    <el-input-number v-model="b" :min="0" :max="num - 1" controls-position="right"
                                        size="large" @change="handleChange" />
                                </div>
                                <el-button type="primary" @click="addEdgeByInput()" size="large" round>添加有向边
                                </el-button>

                                <div class="adjTab">
                                    <table>
                                        <tr v-for="(row, rowIndex) in matrix" :key="rowIndex">
                                            <td v-for="(cell, columnIndex) in row" :key="columnIndex">
                                                <div v-if="rowIndex == columnIndex">
                                                    <el-button type="info" :icon="Message" circle />
                                                </div>
                                                <div v-else>
                                                    <div v-if="cell">
                                                        <el-button type="success" :icon="Check" circle
                                                            @click="changeEdgeByClick(rowIndex, columnIndex, 'success')" />
                                                    </div>
                                                    <div v-else>
                                                        <el-button type="danger" :icon="Delete" circle
                                                            @click="changeEdgeByClick(rowIndex, columnIndex, 'danger')" />
                                                    </div>
                                                </div>
                                            </td>
                                        </tr>
                                    </table>
                                </div>
                            </el-col>
                            <el-col :span="8">
                                <div class="search">
                                    <h1 v-if="que_or_stk === 0"></h1>
                                    <h1 v-else-if="que_or_stk === 1">显示队列</h1>
                                    <h1 v-else-if="que_or_stk === 2">显示栈</h1>
                                    <div v-for="node in queue">
                                        <el-tag>{{ node }}</el-tag>
                                    </div>
                                    <div v-for="node in [...stack].reverse()">
                                        <el-tag>{{ node }}</el-tag>
                                    </div>
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                </div>
                <!-- <div v-for="edge in graphData.links" style="background: aqua">
                    {{ edge.source }} {{ edge.target }}
                </div> -->
            </el-aside>
            <el-container>
                <el-main>
                    <div class="graph-container">
                        <svg id="svg"></svg>
                    </div>
                </el-main>
                <el-footer>
                    <div class="input-data">
                        <div class="input-content">
                            <p>遍历的起始节点</p>
                        </div>
                        <el-input-number v-model="startNode" :min="0" :max="num - 1" @change="handleChange" />
                    </div>
                    <div class="button">
                        <el-button type="success" size="large" round @click="bfs()">开始BFS遍历</el-button>
                    </div>
                    <div class="button">
                        <el-button type="success" size="large" round @click="dfs_recursion()">开始递归的DFS遍历</el-button>
                    </div>
                    <div class="button">
                        <el-button type="success" size="large" round @click="dfs_nonrecursion()">开始非递归的DFS遍历</el-button>
                    </div>
                </el-footer>
            </el-container>
        </el-container>
    </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue' //VE是elmentui，ref是变量，vue是修改g用的
import { ElMessage } from 'element-plus' //发送提示信息
import { Check, Delete, Message } from '@element-plus/icons-vue'
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
const que_or_stk = ref(0)
const queue = ref([])
const stack = ref([])

const color = d3.scaleOrdinal(d3.schemeCategory10);
const width = 1000;
const height = 600;
let originalColors = {}; // 用于存储每个节点的原始颜色


const handleChange = (value) => {
    console.log(value);
}

function numCheckChange(check) {
    matrix.value = Array.from(Array(num.value), () => new Array(num.value).fill(false));
    numCheck.value = check;

    //这里的group是用来确定点的颜色的
    for (let i = 0; i < num.value; i++) {
        graphData.value.nodes.push({ id: i, group: i });
    }

    console.log('matrix Info: ', matrix.value.length, matrix.value[0].length);
}

function addEdgeByInput() {
    if (a.value < 0 || a.value >= num.value || b.value < 0 || b.value >= num.value) {
        ElMessage.error('起始节点和结束节点的编号应该大于0且小于等于' + num.value);
    } else {
        ElMessage({ message: '有向边添加成功.', type: 'success', });
        matrix.value[a.value][b.value] = true;
        graphData.value.links.push({ source: a.value, target: b.value, value: 5 });
        console.log(matrix.value[a.value][b.value]);
    }
}

// 切换边的连接状态
function changeEdgeByClick(rowIndex, columnIndex, type) {
    console.log(`Button at row ${rowIndex} and column ${columnIndex} of type ${type} was clicked.`);
    // 这里你可以根据 rowIndex、columnIndex 和 type 执行更复杂的逻辑
    a.value = rowIndex;
    b.value = columnIndex;
    if (type === 'success') {
        matrix.value[rowIndex][columnIndex] = false;
        matrix.value[columnIndex][rowIndex] = false;

        graphData.value.links = graphData.value.links.filter(edge => {
            return !((edge.source === rowIndex && edge.target === columnIndex)
                || edge.source === columnIndex && edge.target === rowIndex);
        });
    } else {
        matrix.value[rowIndex][columnIndex] = true;
        matrix.value[columnIndex][rowIndex] = true;

        graphData.value.links.push({ source: rowIndex, target: columnIndex, value: 100 });

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


function initGraph(data) {
    // Specify the color scale.

    // The force simulation mutates links and nodes, so create a copy
    // so that re-evaluating this cell produces the same result.
    const links = data.links.map(d => ({ ...d }));
    const nodes = data.nodes.map(d => ({ ...d }));

    // Create a simulation with several forces.
    const simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).strength(0.03))
        .force("charge", d3.forceManyBody().strength(-50))
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
        .attr("stroke-width", 10);

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
        .attr("fill", d => {
            originalColors[d.id] = color(d.group); // 存储原始颜色
            return color(d.group);
        })
        .attr("id", d => "node-" + d.id);

    nodeGroup.append("text")
        .attr("dy", "0.35em")
        .attr("text-anchor", "middle")
        .attr("id", d => "text-" + d.id)
        .attr("class", "unselectable")
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

function bfs() {
    que_or_stk.value = 1;
    stack.value = [];
    queue.value = [];

    for (let i = 0; i < num.value; i++) {
        d3.select(`#text-${i}`).attr("fill", "black");
    }

    let visited = new Array(matrix.value.length).fill(false);

    queue.value.push(startNode.value);
    visited[startNode.value] = true;

    function processNextNode() {
        if (queue.value.length === 0) return; // 结束条件

        let currentNode = queue.value.shift();
        onVisit(currentNode); // Callback to handle node visit

        let currentRow = matrix.value[currentNode];
        if (!currentRow) {
            console.error(`No row in adjMatrix for node ${currentNode}`);
            return;
        }

        for (let i = 0; i < currentRow.length; i++) {
            if (currentRow[i] && !visited[i]) {
                queue.value.push(i);
                visited[i] = true;
            }
        }

        // 使用 setTimeout 递归调用 processNextNode，以添加延迟
        setTimeout(processNextNode, 1000); // 600ms 延迟
    }

    processNextNode(); // 开始 BFS
}

function onVisit(nodeIndex) {
    // 首先，恢复所有节点的原始颜色
    for (let nodeId in originalColors) {
        d3.select(`#node-${nodeId}`).attr("fill", originalColors[nodeId]);
    }

    // 然后，将当前节点的颜色设置为 aqua
    d3.select(`#node-${nodeIndex}`).attr("fill", "aqua");
    d3.select(`#text-${nodeIndex}`).attr("fill", "white");
}

function dfs_recursion() {
    que_or_stk.value = 0;
    stack.value = [];
    queue.value = [];

    let visited = new Array(matrix.value.length).fill(false);

    for (let i = 0; i < num.value; i++) {
        d3.select(`#text-${i}`).attr("fill", "black");
    }

    function recursiveDFS(currentNode, callback = () => {
    }) {
        if (visited[currentNode]) return callback();

        visited[currentNode] = true;
        onVisit(currentNode); // Callback to handle node visit
        console.log(currentNode);

        let currentRow = matrix.value[currentNode];
        if (!currentRow) {
            console.error(`No row in adjMatrix for node ${currentNode}`);
            return callback();
        }

        // 使用一个即时函数来递归遍历邻居
        (function visitNextNeighbor(index) {
            if (index >= currentRow.length) return callback(); // 所有邻居都已被访问

            if (currentRow[index] && !visited[index]) {
                setTimeout(() => {
                    recursiveDFS(index, () => {
                        visitNextNeighbor(index + 1); // 在当前邻居被完全访问后，再访问下一个邻居
                    });
                }, 1000);
            } else {
                visitNextNeighbor(index + 1); // 如果当前邻居已被访问或不存在，直接访问下一个邻居
            }
        })(0); // 从第一个邻居开始
    }


    recursiveDFS(startNode.value); // Start DFS
}

function dfs_nonrecursion() {
    que_or_stk.value = 2;
    stack.value = [];
    queue.value = [];

    let visited = new Array(matrix.value.length).fill(false);

    for (let i = 0; i < num.value; i++) {
        d3.select(`#text-${i}`).attr("fill", "black");
    }

    stack.value.push(startNode.value);
    visited[startNode.value] = true;

    function processNextNode() {
        if (stack.value.length === 0) return; // 结束条件

        let currentNode = stack.value.pop();
        onVisit(currentNode); // Callback to handle node visit

        let currentRow = matrix.value[currentNode];
        if (!currentRow) {
            console.error(`No row in adjMatrix for node ${currentNode}`);
            return;
        }

        for (let i = 0; i < currentRow.length; i++) {
            if (currentRow[i] && !visited[i]) {
                stack.value.push(i);
                visited[i] = true;
            }
        }

        // 使用 setTimeout 递归调用 processNextNode，以添加延迟
        setTimeout(processNextNode, 1000); // 1000ms 延迟
    }

    processNextNode(); // 开始 DFS
}

</script>

<style>
.input-container,
.graph-container {
    font-size: 16px;
    /* 或你想要的其他值 */
    vertical-align: top;
    margin: 30px;
}

.adjTab {
    margin-top: 5px;
}

.el-tag {
    font-size: 50px;
    /* padding: 50px; */
    width: 50px;
    height: 50px;
    margin-top: 5px;
}

.button {
    margin-top: 10px;
    margin-bottom: 20px;
    margin-right: 20px;
    display: inline;
}

.input-data {
    margin-top: 10px;
    margin-bottom: 10px;
}

.input-data .input-content {
    margin-bottom: 2px;
}

.search {
    margin: 50px;
}

.unselectable {
    user-select: none;
    -webkit-user-select: none;
    /* Safari */
    -moz-user-select: none;
    /* Firefox */
    -ms-user-select: none;
    /* IE/Edge */
}
</style>