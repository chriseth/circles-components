<script>
import vis from 'vis-network';
import visData from 'vis-data';
import { onMount } from 'svelte';

const CirclesAPI = 'https://api.circles.garden/api/';
const PathfinderAPI = 'https://pathfinder.exinto.de';

let userDB = {};

let fillUsernames = async function(addresses) {
    // TODO make user info reactive
    let queryString = '';
    for (let addr of addresses) {
        if (addr && !userDB[addr]) {
            queryString += '&address[]=' + addr;
        }
    }
    if (queryString == '') { return; }

    for (let user of (await (await fetch(CirclesAPI + '/users?' + queryString)).json()).data) {
        userDB[user.safeAddress.toLowerCase()] = {avatarUrl: user.avatarUrl, username: user.username};
        nodes.update(small(createNodeContents(user.safeAddress)));
    }
};

let usernameToExplore = '';
let graph;

let nodes = new visData.DataSet();
let edges = new visData.DataSet();
let knownEdges = {};

onMount(() => {
    var network = new vis.Network(graph, {nodes: nodes, edges: edges}, {});
    network.on('click', (params) => {
        if (!params.nodes || params.nodes.length == 0) return;
        exploreNode(params.nodes[0])
    });
});

let toAddress = async function(input) {
    if (input.match(/0x[0-9a-fA-F]{40}/)) {
        return input;
    } else {
        return (await (await fetch(CirclesAPI + '/users?username[]=' + input)).json()).data[0].safeAddress;
    }
};

let labelFor = function(id) {
    let user = userDB[id.toLowerCase()];
    if (user && user.username)
        return user.username;
    else
        return id.substr(0, 8);
};

let createNodeContents = function(id) {
    let user = userDB[id.toLowerCase()];
    let node = {id: id, label: labelFor(id)};
    if (user && user.avatarUrl) {
        node['shape'] = 'circularImage';
        node['image'] = user.avatarUrl;
    }
    return node;
}

let small = function(node) { node['size'] = 10; return node; }
let getAdjacencies = async function(address) {
    return await (await fetch(PathfinderAPI + '/adjacencies/' + address, {mode: 'cors'})).json();
};
let exploreNode = async function(username) {
    let adjacencies = await getAdjacencies(await toAddress(username));
    let addressesToQuery = {};
    for (let edge of adjacencies) {
        addressesToQuery[edge.user] = true;
        addressesToQuery[edge.trusts] = true;
        nodes.update(small(createNodeContents(edge.user)));
        nodes.update(small(createNodeContents(edge.trusts)));
        if (!knownEdges[edge.user + ',' + edge.trusts]) {
            knownEdges[edge.user + ',' + edge.trusts] = true;
            edges.update({
                from: edge.user,
                to: edge.trusts,
                label: edge.percentage == 50 ? '' : edge.percentage + '%',
                arrows: "to",
                font: {align: "top"}
            });
        }
    }
    // async
    fillUsernames(Object.keys(addressesToQuery));
};

let resetGraph = function() {
    nodes.clear();
    edges.clear();
    knownEdges = {};
}

</script>

<main>
    <h1>Adjacency Graph</h1>
    <input bind:value={usernameToExplore} placeholder="username / address">
    <button on:click={exploreNode(usernameToExplore)}>explore</button>
    <button on:click={resetGraph}>reset</button>
    <div bind:this={graph} style="width: 100%; height: 600px;"></div>
</main>

<style>
</style>