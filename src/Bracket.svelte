<script>
    import Bracket from "./Bracket.svelte";
    import Match from "./Match.svelte";

    export let controller;
    export let level = 0;
    export let matchID = 0;

    export let channels = null;
    export let matches;
    export let final = false;
    export let orientation = "left";
</script>

<bracket class="interactive">
    {#if final}
        <leftbracket>
            <Bracket
                bind:controller
                orientation="left"
                level={level - 1}
                {channels}
                matchID={2 * matchID + 2}
                bind:matches
            />
        </leftbracket>
        <final>
            <Match
                bind:controller
                orientation="final"
                {channels}
                {matchID}
                bind:matches
            />
        </final>
        <rightbracket>
            <Bracket
                bind:controller
                orientation="right"
                level={level - 1}
                {channels}
                matchID={2 * matchID + 1}
                bind:matches
            />
        </rightbracket>
    {:else if level !== 0}
        {#if orientation === "right"}
            <current>
                <Match
                    bind:controller
                    {orientation}
                    {channels}
                    {matchID}
                    bind:matches
                />
            </current>
            <previous>
                <Bracket
                    bind:controller
                    orientation="right"
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 2}
                    bind:matches
                />

                <Bracket
                    bind:controller
                    orientation="right"
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 1}
                    bind:matches
                />
            </previous>
        {:else}
            <previous>
                <Bracket
                    bind:controller
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 2}
                    bind:matches
                />

                <Bracket
                    bind:controller
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 1}
                    bind:matches
                />
            </previous>
            <current>
                <Match
                    bind:controller
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
                bind:controller
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
        width: 100%;
        height: fit-content;
        display: flex;
        justify-content: center;
    }
</style>
