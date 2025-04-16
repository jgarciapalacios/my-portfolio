<script>
	import { computePosition, autoPlacement, offset } from '@floating-ui/dom';
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
	import Bar from '$lib/Bar.svelte';
	import StackedBar from '../../lib/StackedBar.svelte';
	import { base } from '$app/paths';
	import FileLines from '../../lib/FileLines.svelte';
	import Scrolly from 'svelte-scrolly';

	let data = [];
	let commits = [];

	let summary = {
		commits: 0,
		files: 0,
		loc: 0,
		maxDepth: 0,
		longestLine: 0,
		maxLinesInFile: 0,
		mostActivePeriod: 'Unknown'
	};

	function getTimeOfDay(hour) {
		if (hour >= 5 && hour < 12) return 'Morning';
		if (hour >= 12 && hour < 17) return 'Afternoon';
		if (hour >= 17 && hour < 21) return 'Evening';
		return 'Night';
	}

	onMount(async () => {
		data = await d3.csv(`${base}/loc.csv`, (row) => ({
			...row,
			line: +row.line,
			depth: +row.depth,
			length: +row.length,
			file: row.file,
			commit: row.commit,
			date: new Date(row.date + 'T00:00' + row.timezone),
			datetime: new Date(row.datetime)
		}));

		commits = d3
			.groups(data, (d) => d.commit)
			.map(([commit, lines]) => {
				let first = lines[0];
				let { author, date, time, timezone, datetime } = first;
				let ret = {
					id: commit,
					url: 'https://github.com/vis-society/lab-7/commit/' + commit,
					author,
					date,
					time,
					timezone,
					datetime,
					hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
					totalLines: lines.length
				};

				Object.defineProperty(ret, 'lines', {
					value: lines,
					configurable: true,
					writable: true,
					enumerable: false
				});

				return ret;
			});

		summary.commits = commits.length;
		summary.files = new Set(data.map((d) => d.file)).size;
		summary.loc = data.length;
		summary.maxDepth = d3.max(data, (d) => d.depth);
		summary.longestLine = d3.max(data, (d) => d.length);
		summary.maxLinesInFile = d3.max(
			Array.from(d3.group(data, (d) => d.file).values()),
			(fileLines) => fileLines.length
		);

		let periodCounts = d3.rollup(
			data,
			(v) => v.length,
			(d) => getTimeOfDay(d.datetime.getHours())
		);

		summary.mostActivePeriod =
			Array.from(periodCounts.entries()).sort((a, b) => b[1] - a[1])[0]?.[0] || 'Unknown';
	});
	let width = 1000,
		height = 600;
	$: minDate = d3.min(commits.map((d) => d.date));
	$: maxDate = d3.max(commits.map((d) => d.date));
	$: maxDatePlusOne = new Date(maxDate);
	$: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

	let commitProgress = 100;
	$: timeScale = d3
		.scaleTime()
		.domain([minDate ?? new Date(2025, 1, 2), maxDate ?? new Date(2025, 4, 2)]) // Feb 2 to May 2
		.range([0, 960]);
	$: commitMaxTime = timeScale.invert(commitProgress);
	$: filteredCommits = commits.filter((commit) => commit.datetime <= commitMaxTime);
	$: filteredLines = data.filter((d) => d.datetime <= commitMaxTime);

	$: xScale = d3
		.scaleTime()
		.domain([minDate, commitMaxTime])
		.range([usableArea.left, usableArea.right])
		.nice();

	$: yScale = d3.scaleLinear().domain([24, 0]).range([usableArea.bottom, usableArea.top]);
	let margin = { top: 10, right: 10, bottom: 30, left: 20 };
	let usableArea = {
		top: margin.top,
		right: width - margin.right,
		bottom: height - margin.bottom,
		left: margin.left
	};
	usableArea.width = usableArea.right - usableArea.left;
	usableArea.height = usableArea.bottom - usableArea.top;

	let xAxis, yAxis;
	$: {
		d3.select(xAxis).call(d3.axisBottom(xScale));
		d3.select(yAxis).call(d3.axisLeft(yScale));
	}
	let yAxisGridlines;
	$: {
		d3.select(yAxisGridlines).call(d3.axisLeft(yScale).tickFormat('').tickSize(-usableArea.width));
	}
	let hoveredIndex = -1;
	$: hoveredCommit = filteredCommits[hoveredIndex] ?? hoveredCommit ?? {};
	let commitTooltip;
	let tooltipPosition = { x: 0, y: 0 };
	let cursor;
	async function dotInteraction(index, evt) {
		let hoveredDot = evt.target;
		if (evt.type === 'mouseenter') {
			hoveredIndex = index;
			cursor = { x: evt.x, y: evt.y };
			tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
				strategy: 'fixed', // because we use position: fixed
				middleware: [
					offset(1), // spacing from tooltip to dot
					autoPlacement()
				]
			});
		} else if (evt.type === 'mouseleave') {
			hoveredIndex = -1;
		} else if (evt.type === 'click') {
			let commit = commits[index];
			if (!clickedCommits.includes(commit)) {
				// Add the commit to the clickedCommits array
				clickedCommits = [...clickedCommits, commit];
			} else {
				// Remove the commit from the array
				clickedCommits = clickedCommits.filter((c) => c !== commit);
			}
		}
	}

	let clickedCommits = [];

	$: allTypes = Array.from(new Set(data.map((d) => d.type)));
	$: selectedLines = (clickedCommits.length > 0 ? clickedCommits : filteredCommits).flatMap(
		(d) => d.lines
	);
	$: selectedCounts = d3.rollup(
		selectedLines,
		(v) => v.length,
		(d) => d.type
	);
	$: languageBreakdown = allTypes.map((type) => [type, selectedCounts.get(type) || 0]);
</script>

<h1>Meta</h1>
<h3>Commit History</h3>
<div class="summary-box">
	<div class="summary-title">Summary</div>
	<div class="stats-grid">
		<div class="stat">
			<div class="stat-label">Commits</div>
			<div class="stat-value">{filteredCommits.length}</div>
		</div>
		<div class="stat">
			<div class="stat-label">Files</div>
			<div class="stat-value">{summary.files}</div>
		</div>
		<div class="stat">
			<div class="stat-label">Total LOC</div>
			<div class="stat-value">{summary.loc}</div>
		</div>
		<div class="stat">
			<div class="stat-label">Max Depth</div>
			<div class="stat-value">{summary.maxDepth}</div>
		</div>
		<div class="stat">
			<div class="stat-label">Longest Line</div>
			<div class="stat-value">{summary.longestLine}</div>
		</div>
		<div class="stat">
			<div class="stat-label">Max Lines</div>
			<div class="stat-value">{summary.maxLinesInFile}</div>
		</div>
		<div class="stat">
			<div class="stat-label">Most Active Time</div>
			<div class="stat-value">{summary.mostActivePeriod}</div>
		</div>
	</div>
</div>
<div class="slider-container">
	<div class="slider-row">
		<label for="time-slider">Show commits up to:</label>
		<input type="range" id="time-slider" min="0" max="960" bind:value={commitProgress} />
	</div>
	<time class="slider-time">
		{commitMaxTime?.toLocaleString()}
	</time>
</div>
<div>
	<svg viewBox="0 0 {width} {height}">
		<g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
		<g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
		<g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
		<g class="dots">
			{#each filteredCommits as commit, index}
				<circle
					on:click={(evt) => dotInteraction(index, evt)}
					on:mouseenter={(evt) => dotInteraction(index, evt)}
					on:mouseleave={(evt) => dotInteraction(index, evt)}
					class:selected={clickedCommits.includes(commit)}
					cx={xScale(commit.datetime)}
					cy={yScale(commit.hourFrac)}
					r="5"
					fill="steelblue"
				/>
			{/each}
		</g>
	</svg>
	<dl
		class="info tooltip"
		bind:this={commitTooltip}
		hidden={hoveredIndex === -1}
		style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"
	>
		<dt>Commit</dt>
		<dd>
			<a href={hoveredCommit.url} target="_blank" rel="noopener noreferrer">{hoveredCommit.id}</a>
		</dd>

		<dt>Author</dt>
		<dd>{hoveredCommit.author}</dd>

		<dt>Date</dt>
		<dd>{hoveredCommit.datetime?.toLocaleDateString('en', { dateStyle: 'full' })}</dd>

		<dt>Time</dt>
		<dd>
			{hoveredCommit.datetime?.toLocaleTimeString('en', { hour: '2-digit', minute: '2-digit' })}
		</dd>

		<dt>Lines Changed</dt>
		<dd>{hoveredCommit.totalLines}</dd>
	</dl>
	<StackedBar data={languageBreakdown} {width} />
</div>
<FileLines lines={filteredLines} {width} />

<style>
	circle {
		transform-origin: center;
		transform-box: fill-box;
		transition: 200ms;

		&:hover {
			transform: scale(1.5);
		}
	}

	.summary-box {
		border: 1px solid #ddd;
		border-radius: 4px;
		padding: 1rem;
		margin: 1rem 0;
		background: white;
		font-family: system-ui, sans-serif;
	}

	.summary-title {
		font-weight: bold;
		font-size: 1.25rem;
		margin-bottom: 1rem;
	}

	.stats-grid {
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;
	}

	.stat {
		text-align: center;
		flex: 1 1 30%;
		min-width: 80px;
		margin: 0.5rem 0;
	}

	.stat-label {
		font-size: 0.75rem;
		color: #667;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.stat-value {
		font-size: 1.5rem;
		font-weight: bold;
		color: #111;
	}

	svg {
		overflow: visible;
	}

	.gridlines {
		stroke-opacity: 0.2;
	}

	.info.tooltip {
		position: absolute;
		background: #fff;
		border: 1px solid #ccc;
		border-radius: 6px;
		padding: 0.75rem 1rem;
		font-size: 0.875rem;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
		color: #333;
		min-width: 220px;
		pointer-events: none;
		z-index: 100;
	}

	.info.tooltip dt {
		font-weight: bold;
		margin-top: 0.5em;
		color: #555;
	}

	.info.tooltip dd {
		margin: 0;
		margin-left: 0.5rem;
		color: #111;
		white-space: nowrap;
	}

	.info.tooltip a {
		color: #0969da;
		text-decoration: none;
		word-break: break-word;
	}

	.info.tooltip a:hover {
		text-decoration: underline;
	}
	.selected {
		fill: var(--color-accent);
	}
	.slider-container {
		display: grid;
		grid-template-rows: auto auto;
		max-width: 100%;
		margin-bottom: 1em;
	}

	.slider-row {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		white-space: nowrap;
	}

	.slider {
		flex: 1;
	}

	.slider-time {
		margin-top: 0.3em;
		font-size: 0.9rem;
		color: #666;
	}
</style>
