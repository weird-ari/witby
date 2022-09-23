<script>
    import Bracket from "./Bracket.svelte";
    import Channel from "./Channel.svelte";

    export let controller;
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
            <Channel
                {controller}
                {channels}
                {matchID}
                bind:matches
                selector={0}
            />
            <Channel
                {controller}
                {channels}
                {matchID}
                bind:matches
                selector={1}
            />
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
            <Channel
                {controller}
                {channels}
                {matchID}
                bind:matches
                selector={0}
            />
            <Channel
                {controller}
                {channels}
                {matchID}
                bind:matches
                selector={1}
            />
        </channels>
        <bracketLine class="left" />
    {:else}
        <finalMatch>
            <finalChannel>
                <verticalLine class="final" />
                <Channel
                    {controller}
                    {channels}
                    {matchID}
                    bind:matches
                    selector={0}
                />
                <verticalLine class="final placeholder" />
            </finalChannel>
            <finalChannel>
                <verticalLine class="final placeholder" />
                <Channel
                    {controller}
                    {channels}
                    {matchID}
                    bind:matches
                    selector={1}
                />
                <verticalLine class="final" />
            </finalChannel>
        </finalMatch>
    {/if}
</match>

<style>
    match {
        height: 100%;
        display: flex;
        justify-content: space-around;
        align-items: center;

        --bracket-line-width: 0.25rem;
        --bracket-line-color: #5a5a63ff;
    }

    channels,
    lines {
        height: 100%;

        display: inline-flex;
        justify-content: space-around;
        flex-direction: column;
    }

    finalMatch {
        display: inline-flex;
        justify-content: space-around;
        flex-direction: column;
    }

    finalChannel {
        height: 100%;

        display: inline-flex;
        justify-content: space-around;
        align-items: center;
        flex-direction: row;
    }

    bracketLine {
        display: inline-block;
        padding: 0;
        margin: 0;

        width: 2rem;
        height: calc(50% - var(--bracket-line-width));

        border: var(--bracket-line-width) solid var(--bracket-line-color);
    }

    bracketLine.left {
        border-left: none;
        border-radius: 0 0.3rem 0.3rem 0;
    }
    bracketLine.right {
        border-right: none;
        border-radius: 0.3rem 0 0 0.3rem;
    }

    verticalLine {
        width: 2rem;
        height: 0rem;
        border-top: var(--bracket-line-width) solid var(--bracket-line-color);
    }

    verticalLine.final {
        width: 2rem;
    }
    verticalLine.placeholder {
        visibility: hidden;
        width: 4rem;
    }
</style>
