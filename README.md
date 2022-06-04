Whilst building my portfolio website I thought it would be cool to add a little memory game for the user to play and decided it could be a great fun project to get some curious programmers to check out this wonderful compiler/framework called **Svelte**.

## Preview of what we're about to build:
[Preview](https://memory-game-tutorial.netlify.app/)

In this tutorial we're going to be use SvelteKit and Tailwind CSS - It's super simple to setup, two commands and you're good to go.

Open your terminal of choice and write this command:

```
npm init svelte your-app-name
```

Svelte is going to ask you a couple of questions, for the purpose of this tutorial you'll want to use TypeScript, ESLINT and Prettier for code formatting and we can skip PlayWright as we're not going do create any tests.


To install tailwind for your project you can just use this command:

```
npx svelte-add@latest tailwindcss
```

#All Set!

Before starting coding it'll be helpful to write down every single thing we expect our program to do, let's sum it up:

- let the player uncover the cards by clicking on them
- only let a max of two cards to be uncovered at any time
- if two cards are uncovered then perform a check
- if the two cards have the same symbol then it's a pair
- if not then cover them up again
- if all the cards are uncovered the player wins

## EDGE CASES
- if a card is already uncovered don't cover it
- if two cards are already uncovered and the user clicks again, do nothing

Writing things down might seem like an extra step but it really helps to visualise all the components you might need and find most of the gotchas that you could encounter.

I'm going to put all the game code in a "Game" component, so inside of the "src" folder I'll create a "lib" folder that will contain all my files, "lib" is useful in SvelteKit because it provides what we call an "alias", meaning that if you have a file that is stored in 
```
"src/lib/components/game/Game.svelte"
```

you can import it by saying:
```javascript
import Game from '$lib/components/game/Game.svelte'
```

Ok so now your "/src/routes/index.svelte" file should look something like this:

```typescript
<script lang="ts">
	import Game from '$lib/components/Game.svelte';
</script>

<Game />

<style>
	:global(body) {
		background-color: rgb(15, 23, 42);
	}
</style>

```

the :global(body) tag is just telling svelte that the current rule applied to "body" is going to be applied to every file in the project -- so for every page the background is going to be 
"rgb(15, 23, 42)"

Now let's create a "Game.svelte" file in "/src/lib/components"

----

By looking at the list of things our program needs to do we can see all the variables we're going to need to implement a basic version of the game so let's create them:

- shouldShow -> helps us with an initial animation on first load
- currentlyUncovered -> helps tracking which entries are uncovered
- isWon -> lets us know if the game is finished or not
- pairedCards -> keeps track of which entries are paired
- cards -> all the cards with their state

translated in code:

```typescript
<script lang="ts">
	let shouldShow: boolean = false;
	let currentlyUncovered: Array<Card> = [];
	let isWon: boolean = false;

	$: pairedCards = cards.filter((f) => f.paired);

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
</script>
```

Let's break this down - if you're unfamiliar with TypeScript just like I am, this is very simple

```let currentlyUncovered: Array<Card> = [];```

"Array<Card>" is basically saying that currentlyUncovered is going to be an Array and it's going to contain elements of type "Card", type "Card" does not exist in javascript, it's not a typical type - but it's a type that we created ourselves a couple of lines below:

```typescript
	interface Card {
		symbol: string;
		paired: boolean;
		covered: boolean;
		id: number;
	}
```
this is saying that our card is an Object and it contains the following keys: "symbol" (which is a string), "paired" (which is a boolean), "covered" and so on..

it's pretty much only declaring the types for every entry in our Card object.

```$: pairedCards = cards.filter((f) => f.paired);```

this line can be quite tricky to understand if you're not familiar with Svelte, "$:" is what we call a "reactive statement"

it's a declaration of a variable (if it's not already declared) which basically says: Everytime anything changes in this expression I'm going to react to it

so in this case everytime the result of ```cards.filter((f) => f.paired)``` is going to change, the value of pairedCards will reflect the result. 

This is **incredibly** powerful and can be a double edged sword as you can imagine how hard it can become to debug reactive statements in a big application if they're abused.

> Quick note on "Math.random()" for our id - for the purpose of this tutorial it's going to work, we only have 16 cards in our array and the likelyhood of an ID to be duplicate is really low.. but if you want to be absolutely sure it's not going to happen you can use a library like [uuid](https://www.npmjs.com/package/uuid) to generate a unique ID.

----

## Markup

All is set for our basic setup and we can move on with creating the basic HTML structure

### Pre requirements
I'm using a library called **[Confetti Explosion](https://www.npmjs.com/package/svelte-confetti-explosion)** to have a nice visual effect when the game is won, it makes the whole experience a little bit more fun. Here is how you install it:

```npm install svelte-confetti-explosion```

and this is how to use it: 

```typescript
					<ConfettiExplosion
						particleCount={200}
						force={0.7}
					/>
```

Now that Confetti Explosion is installed and ready we can move on to creating the markup for the game.

This is what it looks like: (don't worry I'm going to break it down below)

```html

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


```

as you can see the whole block is wrapped in an if statement 

```html
{#if shouldShow}{/if}
```

this is done so we can have a transition when the component is mounted.

right under it the first div that contains the whole game has a transition scale set to it. This is going to execute when we change "shouldShow" from false to true.


### Let's focus on the "each" block now
We are looping through our cards and we need to make sure they have two different states, a covered state and an uncovered one.

We can do this by using CSS transforms and rotate them on the Y axis.

as you can see we have these conditional classes applied to the div

```html
class:uncovered={!card.covered}
class:covered={card.covered}
```

these are "conditional classes" meaning the class "uncovered" will be added to the div if the current card is not covered, and "covered" will be added if the card is covered.

> if you're unfamiliar with loops this is where "card" is coming from: when we opened the each block {#each cards as card} -> "cards" is our object with 16 cards and "card" is the instance we're currently looping through inside of the "each" loop.

it's worth paying attention to this as well:

```html
class="card text-gray-600 cursor-pointer w-32 h-32 {isWon
						? 'opacity-0 pointer-events-none'
						: ''} {card.covered
						? 'bg-gray-800 hover:bg-blue-800 hover:text-white'
						: 'bg-white'} rounded-3xl p-4 relative"
```

if the game is won we are changing the opacity of the cards to 0 and removing any pointer events so the user can't click on them -- this is because we want to keep the game at the same height and the "YOU WON" message at the centre of it (to avoid the container becoming smaller and bouncy); This could have also been done by setting a specific height to the game div but it's just a matter of preference, I found this easier to deal with, especially if the game has to be responsive.

```on:click={() => handleUncovering(card)}``` is just a function that we're going to write soon and it's passing the current card as an argument

note that on events we use an arrow function to pass an argument, if we did 

```html
on:click={handleUncovering(card)}
```

the function would have been called straight away and that's not what we want in this occasion.

Other notes on the markup are:

```html
								in:fade={{ duration: 500, delay: 200 }}
								out:fade={{ duration: 0, delay: 0 }}
```

both the covering and uncovering states have an out transition set to 0 with 0 delay, that's because we want the next animation to start straight away.

Other than that all we're doing is showing the card's symbol if it's uncovered and a question mark if it's covered.

-----

## Game logic

It's finally time to build the game logic - our handleUncovering function will be doing most of the heavy lifting, but before we start we need a way to shuffle our cards like if it was a deck of cards.

I've found a nice algorithm that does exactly that. 

```typescript
	// helper function to shuffle array
	function shuffleArray(array: Array<any>) {
		for (let i = array.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[array[i], array[j]] = [array[j], array[i]];
		}
		// reassign entries to trigger an update
		cards = cards;
	}
```

You can read more about it [here](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#The_modern_algorithm)

So now all we need to do when the component mounts is to shuffle the array and set shouldShow to true, so our component can show, like so

```typescript
	onMount(() => {
		shouldShow = true;
		shuffleArray(cards);
	});
```

Don't forget to import onMount:
```javascript
import { onMount } from 'svelte'
```

> onMount is a Svelte hook, essentially a function that runs when the component is mounted, if you come from react it's equivalent to the old componentDidMount or the new way of doing that with useEffect

Before we start with the big uncovering function we can build two helper functions to:

- Uncover a card
- Reset the currentlyUncovered array

They're not necessary but they'll help reducing repetition in our code

```typescript
  // resets the currently uncovered array
  // and reassign cards to itself so svelte knows
  // it needs to re-render
  function reset() {
    currentlyUncovered = [];
    cards = cards;
  }

  // uncovers an card and reassigns cards to let svelte
  // know to re-render
  function uncover(card: Card) {
    if (card?.covered) {
      card.covered = false;
    }
    cards = cards;
  }
```

Don't worry too much about these at the moment, we've just declared them but we haven't used (called) them yet, we will in a minute

### Now here's the fun part, let's build our handleUncovering function

first let's declare it

```typescript
function handleUncovering(card: Card) {
  // logic here
}
```
Our function will take a card as a parameter with the previously declared type of Card 

Now let's have a look at our list we created earlier. Remember, this function runs everytime the user clicks on **any** card, because we've set the on:click handler on the entry - so let's start doing some checks

```typescript
  // if the card is already uncovered, do nothing
  if (pairedCards.find((f) => f === card)) {
    return;		
  }
```
this checks our pairedCards array, the reactive one, to check if the card we're trying to uncover is already in there, if it is we can stop executing (return)

> returning means stopping the execution of the function, so none of the code inside of the brackets of our function will be executed

let's add another check to deal with an edge case, we want to prevent the user from clicking very quickly to uncover a third card if two of them are already uncovered, so here's how we do it:

```typescript
		// if you've already uncovered two entries do nothing
		if (currentlyUncovered.length === 2) {
			return;
		}
```

Now, if no cards are currently uncovered OR we only uncovered one card we can do some other checks:

```typescript
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
```

This is where we perform the uncovering of cards, if there are 0 or 1 currentlyUncovered cards then we proceed with the uncovering

as you can see we find the one we need to push by using cards.find(), which is an Array method that lets you find elements that match the expression you pass in

so ```const cardToPush: any = cards.find((f) => f.id === card.id)``` will find the element inside of the cards array that has the same id of the one we clicked on, after that's found we use our helper function we created earlier to uncover it

```javascript
uncover(cardToPush);
```

and we also push the uncovered card to the 'currentlyUncovered' array so we can track the uncovered ones:

```javascript
currentlyUncovered.push(card);
```

### All the uncovering is now sorted, we can now deal with mistakes and winning conditions

The possible outcomes at this point are two:

- the cards are matching, in which case we can mark them as "paired"
- the cards don't match, in which case we cover them again

here's the logic:

```typescript
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
```

The comments on this pretty much explain every step, the only thing I'd like to comment on is the timeout of 800ms, that's there just to artificially delay turning over the cards, only to make it clear that a mistake was made and the cards are getting turned over again.

Only thing that is left is to set a winning condition:

```javascript
	// determine if the game is won by checking
	// if the user has paired all the entries
	$: if (pairedCards.length === cards.length) {
		setTimeout(() => {
			isWon = true;
		}, 800);
	}
```

This is a reactive statement that checks the length of our pairedCards array, if the amount of paired cards is the same as the number of total cards then the game is finished, so we can proceed with showing the winning screen by turning isWon to true.


## THAT'S IT! 🥳
Your game is now complete, here's what the code should look like:

Script tag
```typescript
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
```

Markup + Style
```html
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
```

## Thanks for checking this tutorial out!

I hope this helped getting you used to Svelte and Tailwind, I will be doing more SvelteKit tutorials in the form of articles and videos in the coming months so stay tuned for updates!
