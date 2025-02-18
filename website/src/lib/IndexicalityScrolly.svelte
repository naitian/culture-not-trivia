<script>
	import { tick } from 'svelte';
	import { onMount } from 'svelte';
	import scrollama from 'scrollama';
	import Wide from './Wide.svelte';
	import IndexicalField from './IndexicalField.svelte';
	import * as d3 from 'd3';

	const general = {
		from: 'belief',
		to: 'cultural group'
	};
	const firstOrder = {
		from: 'Thinking fizzy, sugary drinks are called "pop"',
		to: 'the Midwest'
	};
	const secondOrder = {
		from: 'Thinking Midwesterners say "pop" instead of "soda"',
		to: 'the U.S.'
	};

	let indices = [general];

	onMount(() => {
		const scroller = scrollama();
		scroller
			.setup({
				step: '.step',
				offset: 0.5,
				progress: true
			})
			.onStepEnter(({ index, element, direction }) => {
				if (index === 0) {
					indices = [general, firstOrder];
				} else if (index === 1) {
					indices = [firstOrder, secondOrder];
				} else if (index === 2) {
				} else if (index === 3) {
				} else {
					lines = [];
				}
			});
	});
</script>

<div class="container">
	<div class="graphic">
		<Wide>
			<IndexicalField {indices} height="200" />
		</Wide>
	</div>

	<div class="step">
		<p>
			Indexicality can occur at different levels of social awareness. A
			<em>first-order</em> index evidences membership in a group. For example, the use of "pop" over
			"soda" might index membership in the population of Midwestern U.S. English speakers.
		</p>
	</div>
	<div class="step">
		<p>
			But higher-order indices occur as these associations themselves become embedded in cultural
			ideology. As such, the knowledge taht Midwesterners say "pop" is <em>in and of itself</em> a piece
			of cultural knowledge.
		</p>
	</div>

    <div class="step"></div>
</div>

<style>
	.container {
		position: relative;
	}

	.graphic {
		position: sticky;
		top: 25vh;
		width: 100%;
		height: 200px;
	}

	.step {
		height: 400px;
		position: relative;
		z-index: -10;
		display: flex;
		align-items: center;
	}
    .step:nth-last-child(1) {
        height: 200px;
    }
	.step p {
		background-color: #f0e9dcbb;
	}
</style>
