<script>
import vis from 'vis-network';
import visData from 'vis-data';
import { onMount } from 'svelte';

import { formatValue } from './utility.js';
import { labelFor, createNodeContents } from './visUtils.js';
import { fillUsernames, fillTokens, tokenDB } from './userdb.js';

export let transfers = [];

let graph;
let nodes = new visData.DataSet();
let edges = new visData.DataSet();
let firstNode = -1;
let lastNode = -1;

onMount(() => {
    let network = new vis.Network(graph, {nodes: nodes, edges: edges}, {});
    network.on('afterDrawing', function() {
        // TODO dimensions of graph?
        let x = window.innerWidth * .4;
        if (firstNode !== -1) {
            nodes.update({id: firstNode, x: -x, y:0});
        }
        if (lastNode !== -1) {
            nodes.update({id:lastNode, x: x, y:0});
        }
    });
});

let retrieveUserAndTokenInfo = async function(steps) {
    let addresses = [];
    let tokens = [];
    for (let step of steps) {
        addresses.push(step.from);
        addresses.push(step.to);
        tokens.push(step.token.toLowerCase());
    }
    await fillUsernames(addresses);
    await fillTokens(tokens);
};

let drawGraph = async function(steps) {
    await retrieveUserAndTokenInfo(steps);
    nodes.clear();
    edges.clear();
    let createNode = function(id) {
        nodes.update(createNodeContents(id));
    };
    for (let step of steps) {
        let tokenUser = tokenDB[step.token.toLowerCase()];
        let label = labelFor(tokenUser || step.token);
        createNode(step.from);
        createNode(step.to);
        edges.update({
            from: step.from,
            to: step.to,
            value: step.value,
            label: formatValue(step.value) + " - " + label,
            arrows: "to",
            font: {align: "top"}
        })
    }
    if (steps.length > 0) {
        firstNode = steps[0].from;
        lastNode = steps[steps.length - 1].to;
    } else {
        firstNode = -1;
        lastNode = -1;
    }
};

$: { drawGraph(transfers) }

</script>

<main>
    <div bind:this={graph} style="width: 100%; height: 600px;"></div>
</main>

<style>
</style>