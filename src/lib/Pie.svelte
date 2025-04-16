<script>
	import * as d3 from 'd3';

	export let data = [];
	export let selectedIndex = -1;

	// Keep arc and pie generator definitions outside reactive context unless they depend on reactive inputs
	const arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
	const sliceGenerator = d3.pie().value((d) => d.value);

	const colors = d3.scaleOrdinal(d3.schemeTableau10);

	let arcData = [];
	let arcs = [];

	$: if (data && data.length) {
		arcData = sliceGenerator(data);
		arcs = arcData.map((d) => arcGenerator(d));
	}
</script>

<div class="container">
	<svg viewBox="-50 -50 100 100">
		{#each arcs as arc, index}
			<path
				d={arc}
				fill={colors(index)}
				class:selected={selectedIndex === index}
				on:click={() => (selectedIndex = index)}
			/>
		{/each}
	</svg>

	<ul class="legend">
		{#each data as d, index}
			<li class:selected={selectedIndex === index}>
				<span class="swatch" style="background-color: {colors(index)};" />
				<span class="label">{d.label}</span>
				<span class="value">({d.value})</span>
			</li>
		{/each}
	</ul>
</div>

<style>
	.container {
		display: flex;
		align-items: center;
	}

	svg {
		max-width: 20em;
		margin: 1em;
	}

	.legend {
		list-style: none;
		padding: 0;
		margin: 0;
	}

	.legend li {
		display: flex;
		align-items: center;
		margin-bottom: 0.5em;
	}

	.swatch {
		width: 1em;
		height: 1em;
		margin-right: 0.5em;
	}

	.label {
		font-weight: bold;
		margin-right: 0.5em;
	}

	.value {
		font-style: italic;
		color: #555;
	}

	svg:has(.selected) path:not(.selected) {
		opacity: 50%;
	}

	.selected {
		--color: oklch(60% 45% 0) !important;

		&:is(path) {
			fill: var(--color) !important;
		}

		&:is(li) {
			color: var(--color);
		}
	}

	ul:has(.selected) li:not(.selected) {
		color: gray;
	}

	path {
		cursor: pointer;
	}
	path:hover {
		opacity: 100% !important;
	}
</style>
