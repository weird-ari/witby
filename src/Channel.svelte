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
                logo: "ytempty.svg",
            };
        } else {
            if (channels[matches[matchID][selector]]) {
                channel = channels[matches[matchID][selector]];
            } else {
                channel = {
                    name: "BYE",
                    logo: "ytempty.svg",
                };
            }
        }
    }

    $: winner = matches[matchID]["winner"] === selector;

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

<channel class:winner on:click={setWinner}>
    <img alt="{channel['name']}}" src={channel["logo"]} />
    <name>
        {channel["name"]}
    </name>
</channel>

<style>
    channel {
        display: block;
        width: 12rem;

        text-align: center;

        border: 0.3rem solid transparent;
        border-radius: 0.3rem;

        background-color: #242429;
        margin: 0.75rem 0;
    }
    channel.winner {
        border-color: #ff9539;
    }

    img {
        width: 100%;
        border-radius: 0.2rem;
        pointer-events: none;
    }

    name {
        display: block;
        color: white;
        font-size: 1.5rem;
        font-family: Anton;
    }
</style>
