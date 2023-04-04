<script lang="ts">
	import type { Pokemon } from '../types/Pokemon.type';

	import pokemonLogo from '$lib/images/pokemon-logo.svg';
	import iPokemon from '$lib/pokemon.json';
	import iEvolutionChains from '$lib/evolution-chains.json';
	import iVersions from '$lib/versions.json';
	import iPokedexes from '$lib/pokedexes.json';
	import type { Pokedex } from '../types/Pokedex.type';
	import type { EvolutionChain } from '../types/EvolutionChain.type';

	let allowLegendaryAndMythical = true;
	let selectedVersionIdx: number = 0;
	let versionPokemon: number[] = [];
	let versionTeam: number[] = [];
	let pokedexes: { [key: string]: Pokedex } = iPokedexes;
	let evolutionChains: { [key: string]: EvolutionChain } = iEvolutionChains;
	let pokemon: { [key: string]: Pokemon } = iPokemon;

	function selectVersion(e: Event) {
		let versionIdx = Number((e.target as HTMLSelectElement).value);
		if (versionIdx > 0) {
			selectedVersionIdx = versionIdx;
			buildTeam();
		}
	}

	function buildTeam() {
		let team = new Set<number>();
		let totalPokemon = new Set<number>();
		let gameEvolutionChains = new Set<number>();
		let totalTypesObj: { [key: string]: number } = {};

		iVersions[selectedVersionIdx].pokedexIds.forEach((pokedexId: number) => {
			pokedexes[pokedexId.toString()].pokemon.forEach((pokemonId) => totalPokemon.add(pokemonId));
		});

		versionPokemon = Array.from(totalPokemon);

		totalPokemon.forEach((pokemonId) => {
			for (let key in evolutionChains) {
				if (evolutionChains[key].pokemon.includes(pokemonId)) {
					gameEvolutionChains.add(evolutionChains[key].evolutionChainId);
				}
			}
		});

		gameEvolutionChains.forEach((chainId) => {
			evolutionChains[chainId.toString()].combinedTypes.forEach((type) => {
				if (iVersions[selectedVersionIdx].types.includes(type)) {
					totalTypesObj[type] = (totalTypesObj[type] || 0) + 1;
				}
			});
		});

		let randomKey = shuffle(Object.keys(totalTypesObj))[0];

		let shuffledEvolutionChains = shuffle(Array.from(gameEvolutionChains));
		while (team.size < 6) {
			for (const chainId of shuffledEvolutionChains) {
				const chainHasType = evolutionChains[chainId].combinedTypes.includes(randomKey);
				const typeIsNotOnTeam = Array.from(team).every((tChainId: any) => tChainId != randomKey);
				const isLegendaryOrMythical = evolutionChains[chainId].pokemon.some(
					(pokemonId) => pokemon[pokemonId].isLegendary || pokemon[pokemonId].isMythical
				);

				if (isLegendaryOrMythical && !allowLegendaryAndMythical) continue;

				if (chainHasType && typeIsNotOnTeam) {
					evolutionChains[chainId].combinedTypes.forEach((type) => delete totalTypesObj[type]);
					team.add(chainId);
					randomKey = shuffle(Object.keys(totalTypesObj))[0];
				}

				if (team.size >= 6) break;
			}
		}

		team.forEach((chainId) => {
			evolutionChains[chainId].pokemon.forEach((pokemonId) => {
				console.log(pokemon[pokemonId].name);
			});
		});

		versionTeam = Array.from(team);
	}

	function shuffle(array: any) {
		let currentIndex = array.length,
			randomIndex;

		// While there remain elements to shuffle.
		while (currentIndex != 0) {
			// Pick a remaining element.
			randomIndex = Math.floor(Math.random() * currentIndex);
			currentIndex--;

			// And swap it with the current element.
			[array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
		}

		return array;
	}
</script>

<svelte:head>
	<title>Pick My Team</title>
</svelte:head>

<section>
	<img class="logo" src={pokemonLogo} alt="pokemon logo" />
	<h1 class="subHeader">Pick My Team</h1>
	<p>
		This app will select your team based on choosing from all Pokemon within a specific game
		version&apos;s Pokedex while also trying to avoid type overlap. The app returns the entire
		evolution chain for that particular Pokemon.
	</p>
	<div>
		<span class="allowLegendaryMythicalOuter">
			<input
				id="allowLegendaryAndMythical"
				class="allowLegendaryAndMythicalInput"
				type="checkbox"
				bind:checked={allowLegendaryAndMythical}
			/>
			<label for="allowLegendaryAndMythical">Allow Legendaries/Mythical Creatures</label>
		</span>
	</div>
	<div>
		<h2>What game are you playing?</h2>
		<select bind:value={selectedVersionIdx} on:change={selectVersion} class="iVersionSelector">
			{#each iVersions as version, i}
				<option value={i}>{version.name}</option>
			{/each}
		</select>
	</div>
	{#if selectedVersionIdx > 0}
		<button class="buildTeam" on:click={buildTeam}>Rebuild Team</button>
		<div>
			{#each versionTeam as chainId}
				<div class="teamContainer">
					{#each evolutionChains[chainId].pokemon as pokemonId}
						<span class="teamItem">
							<img
								src={`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemonId}.png`}
								alt="pokemon"
							/>
							<p>{pokemon[pokemonId].name}</p>
						</span>
					{/each}
				</div>
			{/each}
		</div>
	{/if}
</section>

<style lang="scss">
	:global(body) {
		font-family: Helvetica, sans-serif;
		user-select: none;
		background-image: url('/groovepaper.png');
	}
	.logo {
		display: block;
		margin: 0 auto;
	}
	.subHeader {
		font-family: 'Press Start 2P', cursive;
		text-align: center;
	}
	.typeOuter {
		margin-bottom: 10px;
	}
	.type {
		display: inline-block;
		padding: 5px;
		border: solid thin black;
		box-shadow: 1px 1px 3px;
		border-radius: 100px;
		margin: 0 5px 5px 0;
	}
	.pokemonOuter {
		padding: 10px;
		display: inline-block;
		border: solid thin grey;
		border-radius: 5px;
		margin: 0 10px 10px 0;
	}
	.pokemonImgOuter {
		width: 80px;
		height: 80px;
		background-position: center;
		background-size: contain;
	}
	.pokemonName {
		text-transform: capitalize;
		text-align: center;
		margin: 0;
	}
	.buildTeam {
		margin: 10px 0;
	}
	.teamContainer {
		display: inline-block;
		border: solid 1px slategray;
		border-radius: 4px;
		margin: 5px;
		padding: 10px;
		background-image: url('/dust_scratches.webp');
	}
	.teamItem {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		flex-flow: column;
		padding: 0 8px;
		border-right: solid thin grey;
	}
	.teamItem:last-child {
		margin-right: 0;
		border-right: none;
	}
	.teamItem img {
		width: 70px;
	}
	.allowLegendaryAndMythicalOuter {
		display: inline-flex;
		align-items: center;
	}
	.allowLegendaryAndMythicalOuter label {
		margin-left: 5px;
	}
	.allowLegendaryAndMythicalInput {
		width: 25px;
		height: 25px;
	}
</style>
