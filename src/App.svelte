<script>
	import Match from "./Match.svelte";
	import Bracket from "./Bracket.svelte";

	let channelCount = 8;

	let bracketLevels = Math.ceil(Math.log2(channelCount));

	let channels = [
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
		{
			name: "MrBeast",
			logo: "https://yt3.ggpht.com/ytc/AMLnZu_NBXmT9J0H9uL94tZm6YxOGdMn0utqYJh1aQlv4A=s88-c-k-c0x00ffffff-no-rj",
		},
	];

	fetch("./channels.json")
		.then((res) => res.json())
		.then((out) => {
			channels = out["channels"];
			console.log(channels);
		})
		.catch((err) => console.error(err));

	// https://stackoverflow.com/a/45572051
	var participants = Array.from({ length: channelCount }, (v, k) => k + 1);
	let bracket = getBracket(participants);

	let matches = [];

	for (let i = 0; i < 2 ** (bracketLevels - 1) - 1; i++) {
		matches[i] = {
			0: null,
			1: null,
			winner: undefined,
		};
	}

	for (let i = 0; i < bracket.length; i++) {
		matches[2 ** (bracketLevels - 1) - 1 + i] = {
			0: bracket[i][0] - 1,
			1: bracket[i][1] - 1,
			winner: undefined,
		};
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
	<Bracket final level={bracketLevels - 1} {channels} bind:matches />

	<!--{#each bracket as match, i}
		<Match
			data={{
				channel1: channels[match[0] - 1],
				channel2: channels[match[1] - 1],
				winner: 1,
			}}
		/>
	{/each}-->
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
