<template>
    <div class="common-layout">
        <el-container>
            <el-aside width="400px">
                <div id="input-container">
                    <div v-if="!numCheck">
                        <div class="inputNum-container">
                            <p>请输入图中点的数量</p>
                            <el-input-number v-model="num" :min="1" :max="7" @change="handleChange" />
                            <el-button type="primary" @click="numCheckChange(true)">确认</el-button>
                        </div>
                    </div>
                    <div v-else>
                        <h1>添加有向边</h1>
                        <p>起始节点</p>
                        <el-input-number v-model="a" :min="1" :max="num" controls-position="right" size="large"
                            @change="handleChange" />
                        <p>结束节点</p>
                        <el-input-number v-model="b" :min="1" :max="num" controls-position="right" size="large"
                            @change="handleChange" />
                        <el-button type="primary" @click="addEdgeByInput()" size="large" style="margin-left: 30px">添加有向边
                        </el-button>

                        <div class="adjTab">
                            <table>
                                <tr v-for="(row, rowIndex) in matrix" :key="rowIndex">
                                    <td v-for="(cell, columnIndex) in row" :key="columnIndex">
                                        <div v-if="cell">
                                            <el-button type="success" :icon="Check" circle
                                                @click="changeEdgeByClick(rowIndex, columnIndex, 'success')" />
                                        </div>
                                        <div v-else>
                                            <el-button type="danger" :icon="Delete" circle
                                                @click="changeEdgeByClick(rowIndex, columnIndex, 'danger')" />
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
import { ref, onMounted, watch } from 'vue' //VE是elmentui，ref是变量，vue是修改g用的
import { ElMessage } from 'element-plus' //发送提示信息
import { Check, Delete } from '@element-plus/icons-vue'
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
        nodes.value.push({ id: i })
    }

    edges.value.push({ source: 1, target: 0 });

    console.log(matrix.value.length, matrix.value[0].length);
}

function addEdgeByInput() {
    if (a.value <= 0 || a.value > num.value || b.value <= 0 || b.value > num.value) {
        ElMessage.error('起始节点和结束节点的编号应该大于0且小于等于' + num.value);
    } else {
        ElMessage({ message: '有向边添加成功.', type: 'success', });
        matrix.value[a.value - 1][b.value - 1] = true;
        edges.value.push({ source: a.value - 1, target: b.value - 1 });
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

        edges.value.push({ source: rowIndex, target: columnIndex });
    }
}

//开始画图
// function setupSimulation() {
//     const svg = d3.select('#svg');
//     svg.selectAll("*").remove();  // 清空

//     svg.append("circle")
//         .attr("r", 15)
//         .attr("cx", 100)
//         .attr("cy", 100)
//         .attr("fill", "green");

//     let width = 800;
//     let height = 600;
//     svg.attr('width', width)
//         .attr('height', height);

//     const simulation = d3.forceSimulation(nodes.value)
//         .force("link", d3.forceLink(edges.value).distance(100))
//         .force("charge", d3.forceManyBody().strength(-200))
//         .force("center", d3.forceCenter(width / 2, height / 2));

//     const edgesGraph = svg.append("g")
//         .selectAll("line")
//         .data(edges.value)
//         .enter().append("line")
//         .attr("stroke", "black");

//     const nodesGraph = svg.append("g")
//         .selectAll("circle")
//         .data(nodes.value)
//         .enter().append("circle")
//         .attr("r", 5)
//         .attr("fill", "blue");

//     simulation.on("tick", () => {
//         edgesGraph.attr("x1", d => d.source.x)
//             .attr("y1", d => d.source.y)
//             .attr("x2", d => d.target.x)
//             .attr("y2", d => d.target.y);

//         nodesGraph.attr("cx", d => d.x)
//             .attr("cy", d => d.y);
//     });
// }

// onMounted(() => {
//     setupSimulation();
// });

// watch(edges, () => {
//     setupSimulation();  // re-setup the simulation every time 'edges' changes
// });

const margin = ({ top: 30, right: 80, bottom: 30, left: 30 });
const width = 600, height = 400;

const simulation = d3.forceSimulation()
    .force("link", d3.forceLink() // This force provides links between nodes
        .id(d => d.id) // This sets the node id accessor to the specified function. If not specified, will default to the index of a node.
    )
    .force("charge", d3.forceManyBody().strength(-500)) // This adds repulsion (if it's negative) between nodes. 
    .force("center", d3.forceCenter(width / 2, height / 2)); // This force attracts nodes to the center of the svg area

// const div = html`<div style='max-width: 900px; overflow-x: auto; padding: 0px; margin: 0px;'></div>`;

var tooltip = d3.select("body")
    .append("div")
    .attr("class", "tooltip")
    .style("opacity", 0);

const svg = d3.select("#svg");

svg.attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

//create dummy data
const dataset = {
    nodes: [
        {
            id: 1,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic1.png",
            size: 50
        },
        {
            id: 2,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic2.png",
            size: 35
        },
        {
            id: 3,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic3.png",
            size: 55
        },
        {
            id: 4,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic4.png",
            size: 45
        },
        {
            id: 5,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic5.png",
            size: 52
        },
        {
            id: 6,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic6.png",
            size: 51
        },
        {
            id: 7,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic9.png",
            size: 60
        },
        {
            id: 8,
            img: "https://raw.githubusercontent.com/jienagu/Picture_gif_Personal_Web/master/network_pic10.png",
            size: 60
        }
    ],
    links: [
        { source: 7, target: 1 },
        { source: 7, target: 6 },
        { source: 7, target: 2 },
        { source: 7, target: 3 },
        { source: 7, target: 8 },
        { source: 8, target: 4 },
        { source: 8, target: 5 },
        { source: 8, target: 6 }
    ]
};


// Initialize the links
const link = svg.append("g")
    .attr("class", "links")
    .selectAll("line")
    .data(dataset.links)
    .enter().append("line")
    .style("stroke-width", 2.5);


// Initialize the nodes
// add hover over effect
const node = svg.append("g")
    .attr("class", "nodes")
    .selectAll("image")
    .data(dataset.nodes)
    .enter().append("image")
    .attr("xlink:href", function (d) { return d.img; })
    .attr("width", function (d) { return d.size + 5; })
    .attr("height", function (d) { return d.size + 5; })
    .on("mouseover", function (d) {
        d3.select(this)
            .transition()
            .duration(350)
            .attr("width", 70)
            .attr("height", 70)
    })
    .on("mouseout", function (d) {
        d3.select(this)
            .transition()
            .duration(350)
            .attr("width", function (d) { return d.size; })
            .attr("height", function (d) { return d.size; })
    })
    .on('mouseover.tooltip', function (d) {
        tooltip.transition()
            .duration(300)
            .style("opacity", .8);
        tooltip.html("ID:" + d.id + "<p/>Size:" + d.size)
            .style("left", (d3.event.pageX) + "px")
            .style("top", (d3.event.pageY + 10) + "px");
    })
    .on("mouseout.tooltip", function () {
        tooltip.transition()
            .duration(100)
            .style("opacity", 0);
    })
    .on("mousemove", function () {
        tooltip.style("left", (d3.event.pageX) + "px")
            .style("top", (d3.event.pageY + 10) + "px");
    })
    .call(d3.drag()  //sets the event listener for the specified typenames and returns the drag behavior.
        .on("start", dragstarted) //start - after a new pointer becomes active (on mousedown or touchstart).
        .on("drag", dragged)      //drag - after an active pointer moves (on mousemove or touchmove).
        .on("end", dragended)     //end - after an active pointer becomes inactive (on mouseup, touchend or touchcancel).
    );

//Listen for tick events to render the nodes as they update in your Canvas or SVG.
simulation
    .nodes(dataset.nodes)//sets the simulation’s nodes to the specified array of objects, initializing their positions and velocities, and then re-initializes any bound forces;
    .on("tick", ticked);//use simulation.on to listen for tick events as the simulation runs.
// After this, Each node must be an object. The following properties are assigned by the simulation:
// index - the node’s zero-based index into nodes
// x - the node’s current x-position
// y - the node’s current y-position
// vx - the node’s current x-velocity
// vy - the node’s current y-velocity

simulation.force("link")
    .links(dataset.links)
    .distance(function () { return 100; });


// This function is run at each iteration of the force algorithm, updating the nodes position (the nodes data array is directly manipulated).
function ticked() {
    link.attr("x1", d => d.source.x)
        .attr("y1", d => d.source.y)
        .attr("x2", d => d.target.x)
        .attr("y2", d => d.target.y);

    node.attr("x", d => d.x - 25)
        .attr("y", d => d.y - 25);

}
//create zoom handler 
var zoom_handler = d3.zoom()
    .on("zoom", zoom_actions);

//specify what to do when zoom event listener is triggered 
function zoom_actions() {
    svg.attr("transform", d3.event.transform);
}

//add zoom behaviour to the svg element 
//same as svg.call(zoom_handler); 
zoom_handler(svg);



//When the drag gesture starts, the targeted node is fixed to the pointer
//The simulation is temporarily “heated” during interaction by setting the target alpha to a non-zero value.
function dragstarted(d) {
    if (!d3.event.active) simulation.alphaTarget(0.3).restart();//sets the current target alpha to the specified number in the range [0,1].
    d.fy = d.y; //fx - the node’s fixed x-position. Original is null.
    d.fx = d.x; //fy - the node’s fixed y-position. Original is null.
}

//When the drag gesture starts, the targeted node is fixed to the pointer
function dragged(d) {
    d.fx = d3.event.x;
    d.fy = d3.event.y;
}

//the targeted node is released when the gesture ends
function dragended(d) {
    if (!d3.event.active) simulation.alphaTarget(0);
    d.fx = null;
    d.fy = null;

    //console.log("dataset after dragged is ...",dataset);
}

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

.links line {
    stroke: #000;
}

div.tooltip {
    position: absolute;
    background-color: white;
    max-width: 200px;
    height: auto;
    padding: 1px;
    border-style: solid;
    border-radius: 4px;
    border-width: 1px;
    box-shadow: 3px 3px 10px rgba(0, 0, 0, .5);
    pointer-events: none;
}
</style>