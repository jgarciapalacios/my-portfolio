<script>
	import * as d3 from 'd3';
	import projects from '$lib/projects.json';
	import Project from '$lib/Project.svelte';
	import Pie from '$lib/Pie.svelte';
	$: filteredProjects = projects.filter((project) => {
		if (query) {
			let values = Object.values(project).join('\n').toLowerCase();
			return values.includes(query.toLowerCase());
		}
		return true;
	});

	$: filteredByYear = filteredProjects.filter((project) => {
		if (selectedYear) {
			return project.year === selectedYear;
		}

		return true;
	});

	let pieData;

	$: {
		// Initialize to an empty object every time this runs
		pieData = {};

		// Calculate rolledData and pieData based on filteredProjects here
		let rolledData = d3.rollups(
			filteredProjects,
			(v) => v.length,
			(d) => d.year
		);

		// We don't need 'let' anymore since we already defined pieData
		pieData = rolledData.map(([year, count]) => {
			return { value: count, label: year };
		});
	}
	let query = '';
	let selectedYearIndex = -1;
	let selectedYear;
	$: selectedYear = selectedYearIndex > -1 ? pieData[selectedYearIndex].label : null;
</script>

<svelte:head>
	<title>Projects</title>
</svelte:head>
<Pie data={pieData} bind:selectedIndex={selectedYearIndex} />
<input
	type="search"
	bind:value={query}
	aria-label="Search projects"
	placeholder="🔍 Search projects…"
/>
<h1>{filteredProjects.length} Projects!</h1>
<div class="projects">
	{#each filteredProjects as p}
		<Project data={p} />
	{/each}
</div>
