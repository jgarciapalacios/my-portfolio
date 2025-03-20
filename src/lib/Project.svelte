<script>
	export let data = {};
	export let hLevel = 2;
</script>

<article>
	<svelte:element this={'h' + hLevel}>{data.title}</svelte:element>
	<img src={data.image} alt={data.title} />
	<div>
		<p>{data.description}</p>
		<p class="year">c. {data.year}</p>
	</div>
</article>

<style>
	.projects {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(15em, 1fr));
		gap: 1em;
	}

	.projects article {
		display: grid;
		grid-template-rows: subgrid;
		grid-row-end: span 3;
		margin: 0;
	}

	.year {
		font-style: italic;
	}
</style>
