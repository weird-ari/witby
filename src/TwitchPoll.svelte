<script>
    export let controller;
    export let matchID;
    export let channels;
    export let matches;

    let twitch;
    twitch = new window.tmi.Client({
        channels: ["stanz"],
    });
    twitch.connect().then(() => console.log("CONNECTED"));

    let currentHandler;

    currentHandler = async (channel, tags, message, self) => {
        if (message === "1") {
            matches[matchID]["poll"][0]++;
        } else if (message === "2") {
            matches[matchID]["poll"][1]++;
        }
        console.log(message);
    };
    twitch.on("message", currentHandler);
</script>

<pollFrame>
    <voteCommands>
        <voteCommand class="channel1">1</voteCommand>
        <span class="vs">VS</span>
        <voteCommand class="channel2">2</voteCommand>
    </voteCommands>
    <results>
        <votes class="channel1">{matches[matchID]["poll"][0]}</votes>
        <span>VOTES</span>
        <votes class="channel2">{matches[matchID]["poll"][1]}</votes>
    </results>
    <pollBar>
        <resultBar
            style="width:{(100 * matches[matchID]['poll'][0]) /
                (matches[matchID]['poll'][0] + matches[matchID]['poll'][1])}%"
        />
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
