<script>
    export let controller;
    export let matchID;
    export let matches;

    let ratio;
    $: ratio =
        (100 * matches[matchID]["poll"]["result"][0]) /
        (matches[matchID]["poll"]["result"][0] +
            matches[matchID]["poll"]["result"][1]);

    let twitch;
    let lastChannel;

    $: connectTwitch(controller["settings"]["twitchChannel"]["value"]);

    function connectTwitch(twitchChannel) {
        if (lastChannel === twitchChannel) return;
        if (twitch) {
            console.log(twitch);
            twitch.disconnect();
        }

        console.log(twitchChannel);

        twitch = new window.tmi.Client({
            channels: [twitchChannel.toLowerCase()],
        });
        twitch.connect().then(() => console.log("CONNECTED"));

        let handler = async (channel, tags, message, self) => {
            if (
                matches[matchID]["poll"]["participants"].includes(
                    tags["user-id"]
                )
            )
                return;

            if (message === "1") {
                matches[matchID]["poll"]["result"][0]++;
            } else if (message === "2") {
                matches[matchID]["poll"]["result"][1]++;
            }
            matches[matchID]["poll"]["participants"].push(tags["user-id"]);
            console.log(message, matches[matchID]["poll"]["participants"]);
        };
        twitch.on("message", handler);
        lastChannel = twitchChannel;
    }
</script>

<pollFrame>
    <voteCommands>
        <voteCommand class="channel1">1</voteCommand>
        <span class="vs">VS</span>
        <voteCommand class="channel2">2</voteCommand>
    </voteCommands>
    <results>
        <votes class="channel1">{matches[matchID]["poll"]["result"][0]}</votes>
        <span>VOTES</span>
        <votes class="channel2">{matches[matchID]["poll"]["result"][1]}</votes>
    </results>
    <pollBar>
        <resultBar style="width:{Number.isNaN(ratio) ? 50 : ratio}%" />
    </pollBar>
</pollFrame>

<style>
    pollFrame {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        width: 15rem;
        padding: 1.5rem;
    }
    voteCommands,
    results {
        width: 100%;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
    }
    voteCommand {
        font-size: 6rem;
        padding: 0 1rem;
    }
    votes {
        font-size: 1.5rem;
        padding: 0 1rem;
    }

    span.vs {
        font-size: 2.5rem;
    }

    pollBar {
        display: block;
        width: 100%;
        height: 1rem;
        background-color: #70cedf;
        border-radius: 0.3rem;
    }

    resultBar {
        display: block;
        height: 100%;
        width: 50%;
        background-color: #f69b6c;
        border-radius: 0.3rem 0 0 0.3rem;
    }
</style>
