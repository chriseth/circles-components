<script>
import { toAddress, computeFlow } from './utility';


let transferFrom;
let transferTo;
let transferMaxValue;

export let transfers = [];
export let maxValue = 0;

let compute = async function() {
    let flow = await computeFlow(
        await toAddress(transferFrom),
        await toAddress(transferTo),
        transferMaxValue - 0
    );
    maxValue = flow.flow;
    transfers = flow.transfers;
};

</script>

<main>
    <h1>Computer Transfers</h1>
    <input bind:value={transferFrom} placeholder="from">
    <input bind:value={transferTo} placeholder="to">
    <input bind:value={transferMaxValue} placeholder="max value">
    <button on:click={compute}>compute</button>
</main>

<style>
</style>