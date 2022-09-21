<script>
    export let controller;
    export let channels;
    export let matchID;
    export let matches;
    export let selector;

    let channel, winner;

    $: {
        if (matches[matchID][selector] === null) {
            channel = {
                name: "TBD",
                logo: "ytempty.png",
            };
        } else {
            if (channels[matches[matchID][selector]]) {
                channel = channels[matches[matchID][selector]];
            } else {
                channel = {
                    name: "BYE",
                    logo: "ytempty.png",
                };
            }
        }
    }

    $: winner = matches[matchID]["winner"] === selector;

    function setWinner() {
        if (matches[matchID][0] === null || matches[matchID][1] === null)
            return;

        controller.reset(matchID);

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

<channel class:winner on:click={setWinner}>
    <img alt="{channel['name']}}" src={channel["logo"]} />
    <name>
        {channel["name"]}
    </name>
</channel>

<style>
    channel {
        display: block;
        padding: 0.2rem;
        width: fit-content;
        min-width: 7.5rem;
        text-align: center;
        border: 0.1rem solid transparent;
    }
    channel.winner {
        border-color: black;
    }

    img {
        width: 7.5rem;
    }

    name {
        display: block;
    }
</style>
