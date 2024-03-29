<script>
	import { onMount } from "svelte";
	import FocusedMatch from "./FocusedMatch.svelte";
	import Bracket from "./Bracket.svelte";
	import DataInput from "./DataInput.svelte";

	const getTBDMatch = () => {
		return structuredClone({
			0: null,
			1: null,
			winner: undefined,
			poll: {
				result: [0, 0],
				participants: [],
			},
		});
	};

	let channels = null;

	let bracketLevels;

	let matches = null;

	$: {
		if (matches) {
			localStorage.setItem("WITBYmatches", JSON.stringify(matches));
			console.log("SAVED MATCHES");
		}
	}

	let focusedMatch;

	onMount(async () => {
		const queryString = window.location.search;
		const urlParams = new URLSearchParams(queryString);
		console.log();

		if (urlParams.has("channels") && urlParams.has("ids")) {
			let urlChannels = [];
			let urlChannelNames = urlParams.get("channels").split(",");
			let urlChannelIds = urlParams.get("ids").split(",");
			for (let i = 0; i < urlChannelNames.length; i++) {
				urlChannels.push({
					name: urlChannelNames[i],
					vidID: urlChannelIds[i],
				});
			}
			channels = urlChannels;
		} else {
			const data = await fetch("./channels.json").then((res) =>
				res.json()
			);
			channels = await data["channels"];
		}
		console.log(channels);

		let storageMatches = JSON.parse(localStorage.getItem("WITBYmatches"));

		if (storageMatches && storageMatches.length === channels.length - 1) {
			matches = storageMatches;
			bracketLevels = Math.ceil(Math.log2(channels.length));
			focusedMatch = 2 ** bracketLevels - 2;
			console.log(matches);
		} else {
			controller.setUpBracket(true);
		}
	});

	let controller = {
		settings: {
			twitchChannel: { value: "stanz", name: "Twitch Channel" },
		},
		reset: (matchID) => {
			console.log("RESET", matchID);
			let userConfirm = false;

			let parent = matchID;

			while (parent !== 0) {
				let selector = parent % 2 ? 0 : 1;
				parent = selector ? (parent - 2) / 2 : (parent - 1) / 2;
				if (!userConfirm && matches[parent]["winner"] !== undefined) {
					userConfirm = confirm(
						"You are about to reset the selected and all dependent matches. Proceed?"
					);
					if (!userConfirm) {
						return false;
					}
				}
				if (userConfirm) {
					matches[parent][selector] = null;
					matches[parent]["winner"] = undefined;
					matches[parent]["poll"] = {
						result: [0, 0],
						participants: [],
					};
				}
			}
			return true;
		},
		focusMatch: (id) => {
			focusedMatch = Math.min(Math.max(0, id), matches.length - 1);
		},
		setUpBracket: (skipConfirm) => {
			let confirmed =
				skipConfirm ||
				confirm(
					"Due to changed data a new bracket has to be calculated, which will reset the current bracket. Proceed?"
				);
			if (!confirmed) return false;

			bracketLevels = Math.ceil(Math.log2(channels.length));
			focusedMatch = 2 ** bracketLevels - 2;
			matches = calculateMatches();
			return true;
		},
	};

	function calculateMatches() {
		// https://stackoverflow.com/a/45572051
		var participants = Array.from(
			{ length: channels.length },
			(v, k) => k + 1
		);
		let bracket = getBracket(participants);

		let matches = [];

		for (let i = 0; i < 2 ** (bracketLevels - 1) - 1; i++) {
			matches[i] = getTBDMatch();
		}

		for (let i = 0; i < bracket.length; i++) {
			matches[2 ** (bracketLevels - 1) - 1 + (bracket.length - 1 - i)] = {
				0: bracket[i][1] - 1,
				1: bracket[i][0] - 1,
				winner: undefined,
				poll: {
					result: [0, 0],
					participants: [],
				},
			};
		}

		return matches;
	}

	function getBracket(participants) {
		var participantsCount = participants.length;
		var rounds = Math.ceil(Math.log(participantsCount) / Math.log(2));
		var bracketSize = Math.pow(2, rounds);
		var requiredByes = bracketSize - participantsCount;

		console.log(`Number of participants: ${participantsCount}`);
		console.log(`Number of rounds: ${rounds}`);
		console.log(`Bracket size: ${bracketSize}`);
		console.log(`Required number of byes: ${requiredByes}`);

		if (participantsCount < 2) {
			return [];
		}

		var matches = [[1, 2]];

		for (var round = 1; round < rounds; round++) {
			var roundMatches = [];
			var sum = Math.pow(2, round + 1) + 1;

			for (var i = 0; i < matches.length; i++) {
				var home = changeIntoBye(matches[i][0], participantsCount);
				var away = changeIntoBye(
					sum - matches[i][0],
					participantsCount
				);
				roundMatches.push([home, away]);
				home = changeIntoBye(sum - matches[i][1], participantsCount);
				away = changeIntoBye(matches[i][1], participantsCount);
				roundMatches.push([home, away]);
			}
			matches = roundMatches;
		}
		return matches;
	}

	function changeIntoBye(seed, participantsCount) {
		//return seed <= participantsCount ?  seed : '{0} (= bye)'.format(seed);
		return seed <= participantsCount ? seed : null;
	}
</script>

<main>
	<bracketTitle> Who is the best Youtuber?</bracketTitle>
	<bracketDescription
		>Type <span class="channel1">"1"</span> or
		<span class="channel2">"2"</span> in chat, to vote for the current matchup.</bracketDescription
	>
	<buttonSection>
		<resetButton
			class="interactive"
			on:click={() => controller.setUpBracket()}
		>
			↺
		</resetButton>
	</buttonSection>

	<FocusedMatch
		bind:controller
		matchID={focusedMatch}
		{channels}
		bind:matches
	/>
	{#if matches}
		<Bracket
			bind:controller
			final
			level={bracketLevels - 1}
			{channels}
			bind:matches
		/>
	{:else}
		Loading...
	{/if}
	<DataInput bind:channels bind:controller />
</main>

<style>
	main {
		margin: 0 auto;
		width: fit-content;
		height: fit-content;

		display: flex;
		flex-direction: column;
	}

	bracketTitle {
		font-size: 4rem;
		color: var(--color-cream);
		-webkit-text-stroke: 0.15rem black;
		text-align: center;
		margin: 2rem 0 0.5rem 0;
	}

	bracketDescription {
		font-size: 1.5rem;
		color: white;
		text-align: center;
		margin: 0.25rem 0 1rem 0;
	}

	buttonSection {
		text-align: right;
		margin: 0 2rem 0 0;
	}

	resetButton {
		display: inline-block;
		width: 3rem;
		height: 3rem;
		font-size: 1.9rem;
		text-align: center;
		vertical-align: middle;
		color: white;
		background-color: var(--color-grey-medium);
		border-radius: 0.3rem;
	}
</style>
