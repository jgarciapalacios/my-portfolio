<script>
	import projects from '$lib/projects.json';
	import Project from '$lib/Project.svelte';
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<div class="home">
	<h1>This is my portfolio website, so cool!</h1>
	<p>I like dancing and longboarding and code</p>
	<img src="images/nyc-chess.jpg" alt="Me playing chess in NYC" />
	<h2>Latest Projects</h2>
	<div class="projects">
		{#each projects.slice(0, 3) as p}
			<Project data={p} hLevel="3" />
		{/each}
	</div>
</div>
