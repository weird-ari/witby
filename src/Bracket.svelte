<script>
    import Bracket from "./Bracket.svelte";
    import Match from "./Match.svelte";

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
                orientation="left"
                level={level - 1}
                {channels}
                matchID={2 * matchID + 1}
                bind:matches
            />
        </leftbracket>
        <final>
            <Match {channels} {matchID} bind:matches />
        </final>
        <rightbracket>
            <Bracket
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
                <Match {orientation} {channels} {matchID} bind:matches />
            </current>
            <previous>
                <Bracket
                    orientation="right"
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 1}
                    bind:matches
                />

                <Bracket
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
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 1}
                    bind:matches
                />

                <Bracket
                    level={level - 1}
                    {channels}
                    matchID={2 * matchID + 2}
                    bind:matches
                />
            </previous>
            <current>
                <Match {orientation} {channels} {matchID} bind:matches />
            </current>
        {/if}
    {:else}
        <current>
            <Match leaf {orientation} {channels} {matchID} bind:matches />
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
