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

	let focusedMatch;

	onMount(async () => {
		const data = await fetch("./channels.json").then((res) => res.json());
		channels = await data["channels"];
		controller.setUpBracket(true);
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
	@font-face {
		font-family: Anton;
		src: url("../Anton-Regular.ttf");
		font-weight: bold;
	}
	:global(.channel1) {
		color: #f69b6c;
	}
	:global(.channel2) {
		color: #70cedf;
	}

	:global(body) {
		background-color: #18181b;
		font-family: Anton;
		user-select: none;
	}

	:global(::-webkit-scrollbar) {
		width: 0.5rem;
		height: 0.5rem;
	}

	:global(::-webkit-scrollbar-track) {
		background: #18181b;
	}

	:global(::-webkit-scrollbar-thumb) {
		background: #5a5a63ff;
		border-radius: 0.3rem;
	}

	:global(::-webkit-scrollbar-thumb:hover) {
		background: #5a5a63ff;
	}

	:global(::-webkit-scrollbar-corner) {
		background: transparent;
		width: 0.5rem;
		height: 0.5rem;
	}

	:global(::-webkit-resizer) {
		background: #5a5a63ff;
		width: 0.5rem;
		height: 0.5rem;
	}

	main {
		margin: 0 auto;
		width: fit-content;
		height: fit-content;

		display: flex;
		flex-direction: column;
	}

	bracketTitle {
		font-size: 4rem;
		color: #f5edda;
		-webkit-text-stroke: 0.15rem black;
		text-align: center;
		margin: 3rem 0 0.5rem 0;
	}

	bracketDescription {
		font-size: 1.5rem;
		color: white;
		text-align: center;
		margin: 0.25rem 0 1.5rem 0;
	}
</style>
