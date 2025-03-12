<script>
	import projects from '$lib/projects.json';
	import Project from '$lib/Project.svelte';
	let profileData = {
		ok: true,
		json: async () => ({
			followers: 100,
			following: 100,
			public_repos: 100,
			public_gists: 100
		})
	};
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

<div class="home">
	<h1>This is my portfolio website, so cool!</h1>
	<p>I like dancing and longboarding and code</p>
	<img src="images/nyc-chess.jpg" alt="Me playing chess in NYC" />
	{#await fetch('https://api.github.com/users/jgarciapalacios')}
		<p>Loading...</p>
	{:then response}
		{#await response.json()}
			<p>Decoding...</p>
		{:then data}
			<section>
				<h2>My GitHub Stats</h2>
				<dl>
					<dt>Followers:</dt>
					<dd>{data.followers}</dd>
					<dt>Following:</dt>
					<dd>{data.following}</dd>
					<dt>Public Repositories:</dt>
					<dd>{data.public_repos}</dd>
				</dl>
			</section>
		{:catch error}
			<p class="error">Something went wrong: {error.message}</p>
		{/await}
	{:catch error}
		<p class="error">Something went wrong: {error.message}</p>
	{/await}
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
