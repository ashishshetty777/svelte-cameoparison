<script>
	import {onMount} from 'svelte';
	import Welcome from './screens/Welcome.svelte';
	import Game from './screens/Game.svelte';
	import { select } from './select';
    import { loadImage } from './utils'


	let celebsPromise;
	let state = 'welcome' //alternative is playing;
	let selection;

	const start = async (e) => {
		const {celebs, lookup}  = await celebsPromise;
		selection = select(celebs, lookup, e.detail.category.slug)
		state = 'playing';
	}

	const loadCelebs = async () => {
		const res = await fetch('https://cameo-explorer.netlify.app/celebs.json')
		const data  = await res.json();
		const lookup = new Map();
		const subSet = new Set();

		data.forEach((c) => {
			lookup.set(c.id, c)
		})

		data.forEach((c)=>{
			if(c.reviews >=  50) {
				subSet.add(c)
				c.similar.forEach(id => {
					subSet.add(lookup.get(id))
				})
			}
		})

		return {
			celebs: Array.from(subSet),
			lookup
		}
	}

	onMount(() => {
		celebsPromise = loadCelebs()
		loadImage('/icons/wrong.svg')
		loadImage('/icons/right.svg')
	})

</script>

<main>
	{#if state === 'welcome'}
		<Welcome on:select={start}/>
	{:else if state === 'playing'}
		<Game {selection} on:restart={() => state='welcome'}/>
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 800px;
		height: 100%;
		margin: 0 auto;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
</style>