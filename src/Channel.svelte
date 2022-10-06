<script>
    export let controller;
    export let channels = [];
    export let matchID;
    export let matches;
    export let selector;

    export let focused = false;

    let channel, winner, eliminated;

    $: {
        if (matches[matchID][selector] === null) {
            channel = {
                name: "TBD",
            };
        } else {
            if (channels[matches[matchID][selector]]) {
                channel = channels[matches[matchID][selector]];
            } else {
                channel = {
                    name: "BYE",
                };
            }
        }
    }

    $: winner = matches[matchID]["winner"] === selector;
    $: eliminated = !winner && matches[matchID]["winner"] !== undefined;

    function setWinner() {
        if (matches[matchID][0] === null || matches[matchID][1] === null)
            return;

        let changesConfirmed = controller.reset(matchID);
        if (!changesConfirmed) return;

        if (selector === matches[matchID]["winner"]) {
            matches[matchID]["winner"] = undefined;
            return;
        }

        matches[matchID]["winner"] = selector;

        if (matchID === 0) return;

        let isFirstChild = matchID % 2;
        let parent = isFirstChild ? matchID - 1 : matchID - 2;
        parent = parent / 2;
        matches[parent][isFirstChild ? 0 : 1] = matches[matchID][selector];
        matches[parent]["winner"] = undefined;
    }
</script>

{#if focused}
    <channel class:winner class:focused>
        <img
            class="interactive"
            class:eliminated
            on:click={setWinner}
            alt="{channel['name']}}"
            src={channel["vidID"]
                ? `https://i3.ytimg.com/vi/${channel["vidID"]}/maxresdefault.jpg`
                : "ytempty.svg"}
        />

        <name class="interactive" on:click={setWinner}>
            {channel["name"]}
        </name><a
            href={`https://www.youtube.com/watch?v=${channel["vidID"]}`}
            target="_blank"
        >
            <img class="logo" alt="Youtube logo" src="yt.svg" />
        </a>
    </channel>
{:else}
    <channel class:winner>
        <img
            class:eliminated
            alt="{channel['name']}}"
            src={channel["vidID"]
                ? `https://i3.ytimg.com/vi/${channel["vidID"]}/maxresdefault.jpg`
                : "ytempty.svg"}
        />
        <name>
            {channel["name"]}
        </name>
    </channel>
{/if}

<style>
    channel {
        display: block;
        width: 12rem;

        text-align: center;

        border: 0.2rem solid transparent;
        border-radius: 0.3rem;

        background-color: var(--channel-color, var(--color-grey-medium));
        margin: 0.75rem 0;
    }

    channel:hover {
        --channel-color: var(--color-grey-light);
    }

    channel.winner {
        border-color: var(--color-highlight);
    }

    img {
        width: 100%;
        border-radius: 0.2rem;
    }

    img.eliminated {
        filter: grayscale(100%);
    }

    name {
        display: inline;
        color: white;
        font-size: 1.5rem;
        vertical-align: middle;
        font-family: Anton;
    }

    channel.focused {
        width: 18rem;
    }

    .focused name {
        font-size: 2rem;
    }

    channel .logo {
        display: inline;
        height: 2rem;
        width: auto;
        vertical-align: middle;
        margin: 0 0 0 0.5rem;
    }
</style>
