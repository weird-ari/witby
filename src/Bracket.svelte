<script>
    import Bracket from "./Bracket.svelte";
    import Match from "./Match.svelte";

    export let controller;
    export let level = 0;
    export let matchID = 0;

    export let channels;
    export let matches;
    export let final = false;
    export let orientation = "left";
</script>

<bracket>
    {#if final}
        <leftbracket>
            <Bracket
                {controller}
                orientation="left"
                level={level - 1}
                {channels}
                matchID={2 * matchID + 1}
                bind:matches
            />
        </leftbracket>
        <final>
            <Match {controller} {channels} {matchID} bind:matches />
        </final>
        <rightbracket>
            <Bracket
                {controller}
                orientation="right"
                level={level - 1}
                {channels}
                matchID={2 * matchID + 2}
                bind:matches
            />
        </rightbracket>
    {:else if level !== 0}
        {#if orientation === "right"}
            <current>
                <Match
                    {controller}
                    {orientation}
                    {channels}
                    {matchID}
                    bind:matches
                />
            </current>
            <previous>
                <Bracket
                    {controller}
                    orientation="right"
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 1}
                    bind:matches
                />

                <Bracket
                    {controller}
                    orientation="right"
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 2}
                    bind:matches
                />
            </previous>
        {:else}
            <previous>
                <Bracket
                    {controller}
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 1}
                    bind:matches
                />

                <Bracket
                    {controller}
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 2}
                    bind:matches
                />
            </previous>
            <current>
                <Match
                    {controller}
                    {orientation}
                    {channels}
                    {matchID}
                    bind:matches
                />
            </current>
        {/if}
    {:else}
        <current>
            <Match
                leaf
                {controller}
                {orientation}
                {channels}
                {matchID}
                bind:matches
            />
        </current>
    {/if}
</bracket>

<style>
    bracket {
        width: fit-content;
        height: fit-content;
        display: flex;
        justify-content: space-around;
    }
</style>
