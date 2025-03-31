<script>
	import projects from '$lib/projects.json';
	import Project from '$lib/Project.svelte';
	import { onMount } from 'svelte';

	let githubData = null;
	let loading = true;
	let error = null;

	onMount(async () => {
		try {
			const response = await fetch('https://api.github.com/users/jgarciapalacios');
			githubData = await response.json();
		} catch (err) {
			error = err;
		}
		loading = false;
	});
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<div class="home">
	<h1>This is my portfolio website, so cool!</h1>
	<p>I like dancing and longboarding and code</p>
	<img src="images/nyc-chess.jpg" alt="Me playing chess in NYC" />
	{#if loading}
		<p>Loading...</p>
	{:else if error}
		<p class="error">Something went wrong: {error.message}</p>
	{:else}
		<section>
			<h2>My GitHub Stats</h2>
			<dl>
				<dt>Followers</dt>
				<dd>{githubData.followers}</dd>
				<dt>Following</dt>
				<dd>{githubData.following}</dd>
				<dt>Public Repositories</dt>
				<dd>{githubData.public_repos}</dd>
			</dl>
		</section>
	{/if}
	<h2>Latest Projects</h2>
	<div class="projects">
		{#each projects.slice(0, 3) as p}
			<Project data={p} hLevel="3" />
		{/each}
	</div>
</div>

<style>
	section {
		max-width: 600px;
		margin: auto;
		padding: 1rem;
		border-radius: 8px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
	}

	h2 {
		font-size: 1.5rem;
		margin-bottom: 1rem;
	}

	dl {
		display: grid;
		grid-template-columns: repeat(4, 1fr); /* 4 equal columns */
	}

	dt {
		font-weight: bold;
		font-size: 0.9rem;
		grid-row: 1;
	}

	dd {
		font-size: 1.5rem;
		font-weight: bold;
		margin: 0;
		grid-row: 2;
	}

	.error {
		color: red;
		text-align: center;
	}
</style>
