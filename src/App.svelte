<script>
	import { onMount } from "svelte";
	import Match from "./Match.svelte";
	import Bracket from "./Bracket.svelte";

	const getTBDMatch = () => {
		return structuredClone({
			0: null,
			1: null,
			winner: undefined,
		});
	};

	let channels;

	let bracketLevels;

	let matches;

	onMount(async () => {
		const data = await fetch("./channels.json").then((res) => res.json());
		channels = await data["channels"];

		console.log(channels);
		bracketLevels = Math.ceil(Math.log2(channels.length));
		matches = calculateMatches();
	});

	let controller = {
		reset: (matchID) => {
			console.log("RESET", matchID);
			let parent = matchID;

			while (parent !== 0) {
				let selector = parent % 2 ? 0 : 1;
				parent = selector ? (parent - 2) / 2 : (parent - 1) / 2;
				matches[parent]["winner"] = undefined;
				matches[parent][selector] = null;
			}
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
			matches[2 ** (bracketLevels - 1) - 1 + i] = {
				0: bracket[i][0] - 1,
				1: bracket[i][1] - 1,
				winner: undefined,
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
	{#if matches}
		<Bracket
			{controller}
			final
			level={bracketLevels - 1}
			{channels}
			bind:matches
		/>
	{:else}
		Loading...
	{/if}

	<button
		on:click={() => {
			console.log("click");
		}}>button</button
	>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
