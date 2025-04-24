<script>
	import * as d3 from 'd3';

	export let data = [];
	export let selectedIndex = -1;

	// Keep arc and pie generator definitions outside reactive context unless they depend on reactive inputs
	const arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
	const sliceGenerator = d3.pie().value((d) => d.value);

	$: colors = d3
		.scaleOrdinal()
		.domain(data.map((_, i) => i))
		.range(d3.quantize(d3.interpolateBlues, data.length));

	let arcData = [];
	let arcs = [];

	$: if (data && data.length) {
		arcData = sliceGenerator(data);
		arcs = arcData.map((d) => arcGenerator(d));
	}

	$: description = `A pie chart showing project counts by year. ${data
		.map((d) => `${d.label}: ${d.value} projects`)
		.join(', ')}.`;
	let liveText = '';
	function toggleWedge(index, event) {
		if (!event.key || event.key === 'Enter') {
			selectedIndex = index;
			const d = data[index];
			liveText = `${d.label}: ${d.value} projects selected.`;
		}
	}
	let showChart = true;

	function toggleView() {
		showChart = !showChart;
		liveText = showChart ? 'Pie chart view shown.' : 'Table view shown.';
	}
</script>

<button
	on:click={toggleView}
	aria-pressed={!showChart}
	aria-label="Toggle between pie chart and table view"
	class="toggle-button"
>
	{showChart ? 'Show Table' : 'Show Chart'}
</button>
{#if showChart}
	<div class="container">
		<svg viewBox="-51 -52 105 105" role="img" aria-labelledby="pie-title pie-desc">
			<title id="pie-title">Projects by Year</title>
			<desc id="pie-desc">{description}</desc>
			<circle class="pie-outline" r="50" />
			{#each arcs as arc, index}
				<path
					tabindex="0"
					role="button"
					d={arc}
					fill={colors(index)}
					class:selected={selectedIndex === index}
					on:click={(e) => toggleWedge(index, e)}
					on:keyup={(e) => toggleWedge(index, e)}
					aria-label="wedge {selectedIndex}"
				/>
			{/each}
		</svg>
		<p aria-live="polite" class="sr-only">{liveText}</p>
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
{:else}
	<table aria-label="Table showing project counts by year" class="data-table">
		<caption>Projects by Year</caption>
		<thead>
			<tr>
				<th id="year-header" scope="col">Year</th>
				<th id="projects-header" scope="col">Projects</th>
			</tr>
		</thead>
		<tbody>
			{#each data as d, i}
				<tr>
					<th id="row-{i}" scope="row">{d.label}</th>
					<td aria-labelledby="row-{i} projects-header">{d.value}</td>
				</tr>
			{/each}
		</tbody>
	</table>
{/if}

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
		outline: none;
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
		transition: 300ms;
		outline: none;
	}
	path:hover {
		opacity: 100% !important;
	}
	svg:hover path:not(:hover),
	svg:focus-visible path:not(:focus-visible) {
		opacity: 50%;
	}
	path:focus {
		opacity: 100% !important;
	}
	path:focus-visible {
		stroke: white;
		stroke-width: 2px;
		stroke-dasharray: 4; /* Adjust the dash length as needed */
	}

	.sr-only {
		position: absolute;
		left: -9999px;
		width: 1px;
		height: 1px;
		overflow: hidden;
	}
	.data-table {
		margin-top: 1rem;
		margin-bottom: 1rem;
		border-collapse: collapse;
		width: 100%;
		max-width: 30em;
	}

	.data-table caption {
		font-weight: bold;
		margin-bottom: 0.5em;
		text-align: left;
	}

	.data-table th,
	.data-table td {
		border: 1px solid #ccc;
		padding: 0.5em;
		text-align: left;
	}

	.data-table th {
		background-color: #f0f0f0;
	}
	.pie-outline {
		stroke: black;
		fill: none;
		stroke-width: 1;
	}
</style>
