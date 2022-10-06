<script>
    export let controller;
    export let matchID;
    export let matches;

    let pollStatus = false;

    let lastMatchID;
    $: {
        if (lastMatchID !== matchID) {
            endPoll();
        }
    }

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
            if (!pollStatus) return;

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

    function startPoll() {
        resetPoll();
        pollStatus = true;
    }

    function endPoll() {
        pollStatus = false;
    }

    function resetPoll() {
        matches[matchID]["poll"]["result"] = [0, 0];
        matches[matchID]["poll"]["participants"] = [];
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
    <pollButtons>
        <pollButton on:click={startPoll}>start</pollButton>
        <pollStatus class:active={pollStatus}>
            <img alt="twitch logo" src="twitch.svg" />
        </pollStatus>
        <pollButton on:click={endPoll}>end</pollButton>
    </pollButtons>
</pollFrame>

<style>
    pollFrame {
        display: flex;
        justify-content: space-between;
        align-items: center;
        flex-direction: column;
        height: 100%;
        width: 19rem;
        margin: 0rem 1.5rem;
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
        font-size: 5.5rem;
        padding: 0 1rem;
    }
    votes {
        font-size: 1.5rem;
        padding: 0 1rem;
    }

    span.vs {
        font-size: 2rem;
    }

    pollBar {
        display: block;
        width: 100%;
        height: 1rem;
        background-color: var(--color-blue);
        border-radius: 0.3rem;
    }

    resultBar {
        display: block;
        height: 100%;
        width: 50%;
        background-color: var(--color-orange);
        border-radius: 0.3rem 0 0 0.3rem;
    }

    pollButtons {
        display: flex;
        margin: 0.5rem 0 0 0;
    }

    pollButton,
    pollStatus {
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 0 0.5rem;
        border-radius: 0.3rem;
        height: 2rem;
    }

    pollButton {
        width: 4rem;
        background-color: var(--color-grey-medium);
        font-size: 1.1rem;
        cursor: pointer;
    }

    pollButton:hover {
        background-color: var(--color-grey-light);
    }

    pollStatus {
        width: 2rem;
        background-color: var(--color-grey-medium);
    }

    pollStatus img {
        height: 60%;
        width: 60%;
        filter: grayscale(1);
    }

    pollStatus.active img {
        filter: none;
    }
</style>
