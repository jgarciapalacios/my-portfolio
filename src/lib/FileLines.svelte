<script>
	import * as d3 from 'd3';
	import 'd3-transition';

	export let lines = [];
	export let width = 1200;
	export let colorScale = d3.scaleOrdinal(d3.schemeTableau10);

	let svg;
	const fileInfoHeight = 25;
	const dotRowHeight = 24;
	const dotsColumnX = 280;
	const linesPerDot = 1;
	const approxDotWidth = 13;
	const baseY = 20;
	const totalLinesOffset = 15;

	let previousDotCounts = new Map();

	let files = [];
	$: files = d3
		.groups(lines, (d) => d.file)
		.map(([name, lines]) => ({ name, lines }))
		.sort((a, b) => b.lines.length - a.lines.length);

	$: filesWithHeights = files.map((file) => {
		const totalDots = Math.ceil(file.lines.length / linesPerDot);
		const availableWidth = width - dotsColumnX;
		const maxDotsPerRow = Math.floor(availableWidth / approxDotWidth) || totalDots;
		const dotRows = Math.ceil(totalDots / maxDotsPerRow);
		return { ...file, groupHeight: fileInfoHeight + dotRows * dotRowHeight };
	});

	$: positions = (() => {
		let pos = [],
			y = 0;
		for (const f of filesWithHeights) {
			pos.push(y);
			y += f.groupHeight;
		}
		return pos;
	})();

	function generateDots(file, svgWidth) {
		const totalDots = Math.ceil(file.lines.length / linesPerDot);
		const availableWidth = svgWidth - dotsColumnX;
		const maxDotsPerRow = Math.floor(availableWidth / approxDotWidth) || totalDots;
		const dotRows = Math.ceil(totalDots / maxDotsPerRow);

		let tspans = '';
		for (let r = 0; r < dotRows; r++) {
			const rowLines = file.lines.slice(r * maxDotsPerRow, (r + 1) * maxDotsPerRow);
			const rowDots = rowLines
				.map(
					(line, i) =>
						`<tspan class="dot" data-index="${r * maxDotsPerRow + i}" style="fill:${colorScale(
							line.type
						)}">•</tspan>`
				)
				.join('');
			tspans += `<tspan x="${dotsColumnX}" dy="${r === 0 ? 0 : dotRowHeight}px">${rowDots}</tspan>`;
		}
		return tspans;
	}

	$: if (svg) {
		const totalHeight = positions.length
			? positions[positions.length - 1] + filesWithHeights[filesWithHeights.length - 1].groupHeight
			: 0;

		d3.select(svg).attr('width', width).attr('height', totalHeight).style('overflow', 'hidden');

		const groups = d3
			.select(svg)
			.selectAll('g.file')
			.data(filesWithHeights, (d) => d.name);

		groups.exit().remove();

		const enterGroups = groups
			.enter()
			.append('g')
			.attr('class', 'file')
			.attr('transform', (d, i) => `translate(0, ${positions[i]})`);

		enterGroups
			.append('text')
			.attr('class', 'filename')
			.attr('x', 10)
			.attr('y', baseY)
			.attr('dominant-baseline', 'hanging')
			.text((d) => d.name);

		enterGroups
			.append('text')
			.attr('class', 'linecount')
			.attr('x', 10)
			.attr('y', baseY + totalLinesOffset)
			.attr('dominant-baseline', 'hanging')
			.text((d) => `${d.lines.length} lines`);

		enterGroups
			.append('text')
			.attr('class', 'unit-dots')
			.attr('x', dotsColumnX)
			.attr('y', baseY - 2)
			.attr('dominant-baseline', 'mathematical')
			.attr('fill', '#1f77b4')
			.html((d) => generateDots(d, width));

		// Update selection with transitions
		groups
			.transition()
			.attr('transform', (d, i) => `translate(0, ${positions[i]})`)
			.duration(function (d, i) {
				const currentTransform = this.getAttribute('transform') || 'translate(0,0)';
				const match = currentTransform.match(/translate\(\s*0\s*,\s*([0-9.]+)\s*\)/);
				const oldY = match ? +match[1] : 0;
				const newY = positions[i];
				return Math.abs(newY - oldY) * 2;
			});

		groups.select('text.filename').text((d) => d.name);
		groups
			.select('text.linecount')
			.text((d) => `${d.lines.length} lines`)
			.attr('x', 10)
			.attr('y', baseY + totalLinesOffset);

		groups.each(function (d) {
			const groupSel = d3.select(this);
			const unitDotsSel = groupSel.select('text.unit-dots');
			const newCount = d.lines.length;
			const oldCount = previousDotCounts.get(d.name) || 0;

			unitDotsSel.html(generateDots(d, width));

			if (newCount > oldCount) {
				unitDotsSel
					.selectAll('tspan.dot')
					.filter(function () {
						return +this.getAttribute('data-index') >= oldCount;
					})
					.style('opacity', 0)
					.transition()
					.duration(1000)
					.ease(d3.easeCubicOut)
					.style('opacity', 1);
			}

			previousDotCounts.set(d.name, newCount);
		});
	}
</script>

<svg bind:this={svg} />

<style>
	:global(text.filename) {
		font-size: 0.5rem;
		font-weight: bold;
		font-family: monospace;
	}

	:global(text.linecount) {
		font-size: 0.5rem;
		fill: #666;
	}

	:global(text.unit-dots) {
		font-size: 1.6rem;
	}
</style>
