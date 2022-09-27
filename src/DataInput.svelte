<script>
    export let channels;
    export let controller;

    let expanded = false;

    let jsonString;
    let settingsJsonString;
    let textarea;
    let settingsTextarea;

    $: jsonString = JSON.stringify(channels, null, 5);
    $: settingsJsonString = JSON.stringify(controller["settings"], null, 5);

    function importJSON(area) {
        channels = JSON.parse(area.value);
    }

    function addChannel() {
        channels = [
            ...channels,
            {
                name: "",
                vidID: "",
            },
        ];

        if (
            Math.ceil(Math.log2(channels.length)) >
            Math.ceil(Math.log2(channels.length - 1))
        ) {
            if (!controller.setUpBracket()) {
                channels.splice(channels.length - 1);
            }
        }
    }

    function removeChannel(i) {
        let oldChannels = channels;
        channels = [...channels.slice(0, i), ...channels.slice(i + 1)];
        if (
            Math.ceil(Math.log2(channels.length)) <
            Math.ceil(Math.log2(channels.length + 1))
        ) {
            if (!controller.setUpBracket()) {
                channels = oldChannels;
            }
        }
    }
</script>

<toggleButton
    class="interactive"
    on:click={(e) => {
        expanded = !expanded;
        if (expanded) {
            e.target.innerHTML = "▼ Settings";
        } else {
            e.target.innerHTML = "▶ Settings";
        }
    }}>▶ Settings</toggleButton
>
<dataInput>
    {#if expanded}
        <dataLabel>Channels</dataLabel>
        <section>
            <formBlock>
                {#if channels}
                    {#each channels as { name, vidID }, i}
                        <channel>
                            <channelButton
                                class="delete"
                                on:click={() => {
                                    removeChannel(i);
                                }}
                            >
                                ⌦
                            </channelButton>
                            <input
                                type="text"
                                class="channel-name"
                                bind:value={name}
                            />
                            <input
                                type="text"
                                class="video-id"
                                bind:value={vidID}
                            />
                        </channel>
                    {/each}
                {/if}
                <channel>
                    <channelButton class="add" on:click={addChannel}
                        >＋</channelButton
                    >
                </channel>
            </formBlock>
            <jsonBlock>
                <textarea
                    bind:this={textarea}
                    class="json"
                    value={jsonString}
                    on:input={() => {
                        importJSON(textarea);
                    }}
                />
            </jsonBlock>
        </section>
        <dataLabel>Settings</dataLabel>
        <section>
            <formBlock>
                {#each Object.entries(controller["settings"]) as [setting, data], i}
                    <setting>
                        <settingLabel>
                            {data["name"]}:
                        </settingLabel>
                        <input
                            type="text"
                            class="channel-name"
                            value={data["value"]}
                            on:change={(e) => {
                                data["value"] = e.target.value;
                            }}
                        />
                    </setting>
                {/each}
            </formBlock>
            <jsonBlock>
                <textarea
                    bind:this={settingsTextarea}
                    class="json"
                    value={settingsJsonString}
                    on:input={() => {
                        importJSON(settingsTextarea);
                    }}
                />
            </jsonBlock>
        </section>
    {/if}
</dataInput>

<style>
    toggleButton {
        width: fit-content;
        color: var(--color-grey-light);
        font-size: 1.5rem;
        margin: 5rem 0.5rem 2rem 0;
        text-decoration: underline;
    }

    dataInput {
        width: 100%;
    }

    dataLabel {
        color: white;
        font-size: 1.5rem;
        padding: 0.5rem;
        margin-top: 1.5rem;
    }

    settingLabel {
        font-family: sans-serif, Helvetica, Arial;
        color: white;
        font-size: 1rem;
        margin-top: 1rem;
    }

    channelButton {
        display: inline-block;
        width: 2rem;
        height: 2rem;
        margin: 0.25rem;

        border-radius: 0.3rem;

        font-size: 1.3rem;
        background-color: var(--color-grey-medium);
        color: white;
        text-align: center;
        vertical-align: middle;

        cursor: pointer;
    }

    input,
    textarea {
        font-family: sans-serif, Helvetica, Arial;
        color: white;

        vertical-align: middle;

        margin: 0.2rem;
        background-color: var(--color-grey-medium);

        border: 0;
        border-radius: 0.4rem;
        resize: none;
    }

    input:focus-visible,
    textarea:focus-visible {
        outline: 0.1rem solid var(--color-highlight);
    }

    channel {
        display: flex;
        align-items: center;
    }

    input.channel-name {
        font-weight: bold;
    }

    section {
        display: flex;
        margin-bottom: 1.5rem;
    }

    formBlock,
    jsonBlock {
        display: inline-block;
        padding: 0.5rem;
    }

    formBlock {
        width: fit-content;
    }

    jsonBlock {
        min-height: 10rem;
        flex-grow: 1;
    }

    .json {
        width: 100%;
        height: calc(100% - 2rem);
    }
</style>
