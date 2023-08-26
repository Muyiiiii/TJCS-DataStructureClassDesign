<template>
    <svg id="svg" width="800" height="600" style="background-color: brown;"></svg>
</template>

<script setup>
import * as d3 from 'd3';
import {ref, onMounted} from 'vue';


onMounted(() => {
    // const svg = d3.select('#svg');
    // const g = svg.append("g"); //整个图的group
    //
    // g.append('circle')
    //     .attr('cx', 100)
    //     .attr('cy', 100)
    //     .attr('r', 50)
    //     .attr('fill', 'green');

    // const svg = d3.select('#svg');
    // const g = svg.append("g"); //整个图的group
    //
    // svg.attr('width', 800)
    //     .attr('height', 600);
    //
    // let width = 800;
    // let height = 600;
    // let yStep = 100;
    // let xMargin = 10;
    // let yMargin = yStep;
    // let y = yMargin;
    //
    // for (var i = 1; i <= (height / yStep); i++) {
    //     for (var j = 1; j < Math.pow(2, i); j++) {
    //         var x = (width / Math.pow(2, i)) * (2 * j - 1)
    //         g.append("line")
    //             .attr("x1", x)
    //             .attr("x2", x + (width / Math.pow(2, i)) / 2)
    //             .attr("y1", y)
    //             .attr("y2", y + yStep)
    //             .attr("stroke", "#eba834")
    //             .attr("stroke-width", 3)
    //
    //         g.append("line")
    //             .attr("x1", x)
    //             .attr("x2", x - (width / Math.pow(2, i)) / 2)
    //             .attr("y1", y)
    //             .attr("y2", y + yStep)
    //             .attr("stroke", "#eba834")
    //             .attr("stroke-width", 3);
    //
    //         g.append("circle")
    //             .attr("cx", x)
    //             .attr("cy", y)
    //             .attr("r", 10)
    //             .attr("fill", "green")
    //             .attr("stroke-width", 2);
    //     }
    //     y += yStep;
    // }

    const matrix = [
        [0, 1, 0],
        [1, 0, 1],
        [0, 1, 0]
    ];

    const nodes = [];
    const links = [];

    for (let i = 0; i < matrix.length; i++) {
        nodes.push({id: i});
        for (let j = 0; j < matrix[i].length; j++) {
            if (matrix[i][j] === 1) {
                links.push({source: i, target: j});
            }
        }
    }

    const svg = d3.select('svg');
    const width = +svg.attr('width');
    const height = +svg.attr('height');

    const simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).distance(100))
        .force("charge", d3.forceManyBody().strength(-200))
        .force("center", d3.forceCenter(width / 2, height / 2));

    const link = svg.append("g")
        .selectAll("line")
        .data(links)
        .enter().append("line")
        .attr("stroke", "black");

    const node = svg.append("g")
        .selectAll("circle")
        .data(nodes)
        .enter().append("circle")
        .attr("r", 5)
        .attr("fill", "blue");

    simulation.on("tick", () => {
        link.attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y);

        node.attr("cx", d => d.x)
            .attr("cy", d => d.y);
    });

});


</script>

<style></style>
