<script>
    import Bracket from "./Bracket.svelte";
    import Channel from "./Channel.svelte";

    export let channels;
    export let matches;
    export let matchID;
    export let orientation;
    export let leaf = false;
</script>

<match>
    {#if orientation === "right"}
        <bracketLine class="right" />
        <channels>
            <Channel {channels} {matchID} bind:matches selector={0} />
            <Channel {channels} {matchID} bind:matches selector={1} />
        </channels>
        {#if !leaf}
            <lines>
                <verticalLine />
                <verticalLine />
            </lines>
        {/if}
    {:else if orientation === "left"}
        {#if !leaf}
            <lines>
                <verticalLine />
                <verticalLine />
            </lines>
        {/if}
        <channels>
            <Channel {channels} {matchID} bind:matches selector={0} />
            <Channel {channels} {matchID} bind:matches selector={1} />
        </channels>
        <bracketLine class="left" />
    {:else}
        <verticalLine />
        <Channel {channels} {matchID} bind:matches selector={0} />
        <Channel {channels} {matchID} bind:matches selector={1} />
        <verticalLine />
    {/if}
</match>

<style>
    match {
        height: 100%;
        display: flex;
        justify-content: space-around;
        align-items: center;
    }

    channels,
    lines {
        height: 100%;

        display: inline-flex;
        justify-content: space-around;
        flex-direction: column;
    }

    bracketLine {
        display: inline-block;
        width: 2rem;
        height: 50%;
        border: 0.2rem solid black;
        padding: 0;
        margin: 0;
    }

    bracketLine.left {
        border-left: none;
    }
    bracketLine.right {
        border-right: none;
    }

    verticalLine {
        width: 2rem;
        height: 0rem;
        border-top: 0.2rem solid black;
    }
</style>
