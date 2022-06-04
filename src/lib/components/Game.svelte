<script lang="ts">
	// import { v4 as uuidv4 } from 'uuid';
	import { onMount } from 'svelte';
	import { fade, scale } from 'svelte/transition';
	import { ConfettiExplosion } from 'svelte-confetti-explosion';

	let currentlyUncovered: Array<Card> = [];
	let isWon: boolean = false;
	let shouldShow: boolean = false;

	// reactive statement to determine how many
	// entries the user has paid
	$: pairedCards = cards.filter((f) => f.paired);

	// determine if the game is won by checking
	// if the user has paired all the entries
	$: if (pairedCards.length === cards.length) {
		setTimeout(() => {
			isWon = true;
		}, 800);
	}

	interface Card {
		symbol: string;
		paired: boolean;
		covered: boolean;
		id: number;
	}

	let cards: Array<Card> = [
		{ symbol: '👻', paired: false, covered: true, id: Math.random() },
		{ symbol: '💩', paired: false, covered: true, id: Math.random() },
		{ symbol: '🐊', paired: false, covered: true, id: Math.random() },
		{ symbol: '🐳', paired: false, covered: true, id: Math.random() },
		{ symbol: '🦥', paired: false, covered: true, id: Math.random() },
		{ symbol: '🍄', paired: false, covered: true, id: Math.random() },

		{ symbol: '👻', paired: false, covered: true, id: Math.random() },
		{ symbol: '💩', paired: false, covered: true, id: Math.random() },
		{ symbol: '🐊', paired: false, covered: true, id: Math.random() },
		{ symbol: '🐳', paired: false, covered: true, id: Math.random() },
		{ symbol: '🦥', paired: false, covered: true, id: Math.random() },
		{ symbol: '🍄', paired: false, covered: true, id: Math.random() }
	];

	// helper function to shuffle array
	function shuffleArray(array: Array<any>) {
		for (let i = array.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[array[i], array[j]] = [array[j], array[i]];
		}
		// reassign entries to trigger an update
		cards = cards;
	}

	function handleUncovering(card: Card) {
		// if the card is already uncovered, do nothing
		if (pairedCards.find((f) => f === card)) {
			return;
		}
		// if you've already uncovered two entries do nothing
		if (currentlyUncovered.length === 2) {
			return;
		}
		// if there is already an uncovered card and the user clicks on the same one again
		// do nothing
		if (!currentlyUncovered.length || currentlyUncovered.length === 1) {
			if (card === currentlyUncovered[0]) {
				return;
			}
			// find the card the user has clicked on
			const cardToPush: any = cards.find((f) => f.id === card.id);
			// uncover the card and push it to the currently uncovered array
			uncover(cardToPush);
			currentlyUncovered.push(card);
		}

		// if the user uncovers two entries then start doing checks
		if (currentlyUncovered.length === 2) {
			// if the two entries have the same symbol then they're a match
			if (currentlyUncovered[0].symbol === currentlyUncovered[1].symbol) {
				// loop through them and change their state to 'paired'
				currentlyUncovered.forEach((f) => {
					f.paired = true;
				});

				reset();
			} else {
				// if it's not a match then loop through the uncovered
				// entries and cover them back again
				// timeout to not make it instant, let the user
				// realise they made a mistake
				setTimeout(() => {
					currentlyUncovered.forEach((f) => {
						f.covered = true;
					});

					reset();
				}, 800);
			}
		}
	}
	// resets the currently uncovered array
	// and reassign entries to itself so svelte knows
	// it needs to re-render
	function reset() {
		currentlyUncovered = [];
		cards = cards;
	}

	// uncovers an card and reassigns entries to let svelte
	// know to re-render
	function uncover(card: Card) {
		if (card?.covered) {
			card.covered = false;
		}
		cards = cards;
	}

	onMount(() => {
		shouldShow = true;
		shuffleArray(cards);
	});

	function playAgain() {
		cards.forEach((f) => {
			f.paired = false;
			f.covered = true;
		});
		pairedCards = [];
		isWon = false;
		cards = cards;
	}
</script>

{#if shouldShow}
	<div
		transition:scale={{ duration: 400 }}
		class="game absolute w-max z-50 p-8 bg-gray-900 shadow-3xl rounded-3xl text-white top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2"
	>
		<h2 class="text-center font-extrabold mb-8 text-3xl text-gray-200 tracking-tighter">Memory</h2>
		<div class="game-grid relative grid grid-cols-3 gap-1">
			{#if isWon}
				<div class="fixed top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">
					<ConfettiExplosion particleCount={200} force={0.7} />
				</div>
				<div
					class="absolute w-full top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2  confetti-explosion mx-auto text-center py-16"
				>
					<div class="win-message flex flex-wrap justify-center">
						<p class="text-2xl w-full font-light tracking-tighter italic mb-0">congratulations</p>
						<h2 class="text-6xl w-full font-black tracking-tighter mb-4">YOU WON!</h2>
						<div class="flex gap-3">
							<button
								class="text-white bg-transparent border border-white hover:bg-white hover:text-black transition-all py-1 px-4 rounded-md cursor-pointer w-max"
								on:click={playAgain}>Play Again</button
							>
						</div>
					</div>
				</div>
			{/if}

			{#each cards as card}
				<div
					class:uncovered={!card.covered}
					class:covered={card.covered}
					class="card text-gray-600 cursor-pointer w-32 h-32 {isWon
						? 'opacity-0 pointer-events-none'
						: ''} {card.covered
						? 'bg-gray-800 hover:bg-blue-800 hover:text-white'
						: 'bg-white'} rounded-3xl p-4 relative"
					on:click={() => handleUncovering(card)}
				>
					<div
						class="absolute symbol top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 text-6xl"
					>
						{#if card.covered}
							<span
								in:fade={{ duration: 500, delay: 200 }}
								out:fade={{ duration: 0, delay: 0 }}
								class="text-6xl font-black"
							>
								?
							</span>
						{:else}
							<div
								in:fade={{ duration: 500, delay: 200 }}
								out:fade={{ duration: 0, delay: 0 }}
								class="symbol"
							>
								{card.symbol}
							</div>
						{/if}
					</div>
				</div>
			{/each}
		</div>
	</div>
{/if}

<style>
	.uncovered {
		transform: rotateY(180deg);
	}
	.card {
		transition: 0.8s all ease;
	}
	.covered {
		transform: rotateY(0deg);
	}
</style>
