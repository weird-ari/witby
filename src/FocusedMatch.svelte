<script>
    import Channel from "./Channel.svelte";
    import TwitchPoll from "./TwitchPoll.svelte";

    export let matchID;
    export let controller;

    export let channels;
    export let matches;
</script>

<match>
    {#if matches}
        <nav
            class="interactive"
            on:click={() => controller.focusMatch(matchID + 1)}
        >
            ←
        </nav>
        <Channel
            focused
            {controller}
            {channels}
            {matchID}
            bind:matches
            selector={1}
        />
        <TwitchPoll bind:controller {matchID} bind:matches />
        <Channel
            focused
            {controller}
            {channels}
            {matchID}
            bind:matches
            selector={0}
        />
        <nav
            class="interactive"
            on:click={() => controller.focusMatch(matchID - 1)}
        >
            →
        </nav>
    {/if}
</match>

<style>
    match {
        height: fit-content;
        text-align: center;
        color: white;

        display: flex;
        justify-content: center;
        align-items: center;

        margin: 0.5rem 0 1.5rem 0;
    }

    nav {
        padding: 0.2rem 1rem;
        border-radius: 0.3rem;
        margin: 2rem;
        font-size: 1.5rem;
        background-color: var(--color-grey-medium);
    }

    nav:hover {
        background-color: var(--color-grey-light);
    }
</style>
