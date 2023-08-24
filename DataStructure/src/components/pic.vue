<template>
    <svg id="svg" style="background-color: brown;"></svg>
</template>

<script setup>
import * as d3 from 'd3';
import { ref, onMounted } from 'vue';


onMounted(() => {
    const svg = d3.select('#svg');
    const g = svg.append("g"); //整个图的group

    g.append('circle')
        .attr('cx', 100)
        .attr('cy', 100)
        .attr('r', 50)
        .attr('fill', 'green');
});


</script>

<style></style>
