<script>
	import * as d3 from 'd3';

	export let data = []; // Expected: [ [label, count], [label2, count2], ... ]
	export let width = 400;
	export let barHeight = 30;

	let selectedIndex = -1;
	let hoveredIndex = -1;
	const MIN_LABEL_WIDTH = 40;

	// Extract labels only once
	$: keys = data.map(([label]) => label);

	// Transform to { label1: count1, label2: count2, ... } and wrap in array
	$: dataForStack = [Object.fromEntries(data)];

	// Stack the data
	$: stackedData = d3.stack().keys(keys)(dataForStack);

	// Total max value for scale
	$: total = d3.max(stackedData, (series) => d3.max(series, (d) => d[1])) || 1;

	$: xScale = d3.scaleLinear().domain([0, total]).range([0, width]);

	$: colorScale = d3.scaleOrdinal().domain(keys).range(d3.schemeTableau10);
</script>

<div class="container">
	<svg {width} height={barHeight}>
		{#each stackedData as series, i (series.key)}
			{#each series as d, j}
				<rect
					class:selected={selectedIndex === i}
					class:hovered={hoveredIndex === i}
					x={xScale(d[0])}
					y="0"
					width={xScale(d[1]) - xScale(d[0])}
					height={barHeight - 5}
					fill={colorScale(series.key)}
					on:click={() => (selectedIndex = selectedIndex === i ? -1 : i)}
					on:mouseenter={() => (hoveredIndex = i)}
					on:mouseleave={() => (hoveredIndex = -1)}
				/>
				{#if xScale(d[1]) - xScale(d[0]) > MIN_LABEL_WIDTH}
					<text
						class="label"
						x={(xScale(d[0]) + xScale(d[1])) / 2}
						y={barHeight / 2}
						text-anchor="middle"
						fill="white"
						dominant-baseline="middle"
					>
						{series.key}: {d[1] - d[0]}
					</text>
				{/if}
			{/each}
		{/each}
	</svg>

	<ul class="legend">
		{#each stackedData as series, i (series.key)}
			<li
				style="--color: {colorScale(series.key)}"
				class:selected={selectedIndex === i}
				on:click={() => (selectedIndex = selectedIndex === i ? -1 : i)}
			>
				<span class="swatch" />
				{series.key} <em>({series[0][1] - series[0][0]})</em>
			</li>
		{/each}
	</ul>
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	rect {
		transition: all 300ms ease;
		cursor: pointer;
	}
	rect.hovered {
		stroke-width: 2;
	}
	svg:has(rect.hovered) rect:not(.hovered) {
		opacity: 0.3;
	}
	rect.selected {
		stroke-width: 2;
	}
	svg:has(rect.selected) rect:not(.selected) {
		opacity: 0.3;
	}
	rect.selected.hovered {
		stroke-width: 3;
		opacity: 1;
	}
	.label {
		font-size: 0.75em;
		pointer-events: none;
	}
	.legend {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(8em, 1fr));
		gap: 0.5em;
		list-style: none;
		padding: 1em;
		margin: 1em 0;
		border: 1px solid;
		background: transparent;
		width: 100%;
		box-sizing: border-box;
		text-align: left;
	}
	.legend li {
		color: var(--color);
		cursor: pointer;
		font-size: 0.85em;
	}
	.legend li.selected {
		font-weight: bold;
	}
	.legend:has(.selected) li:not(.selected) {
		color: gray;
		opacity: 0.5;
	}
	.legend li .swatch {
		background: var(--color);
		display: inline-block;
		width: 1em;
		height: 1em;
		border-radius: 0.2em;
		margin-right: 0.25em;
	}
	.legend em {
		font-style: normal;
		color: gray;
	}
</style>
