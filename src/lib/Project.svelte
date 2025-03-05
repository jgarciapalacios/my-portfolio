<script>
	export let data = {};
	export let hLevel = 2;
</script>

<article>
	<svelte:element this={'h' + hLevel}>{data.title}</svelte:element>
	<p>{data.description}</p>
	<img src={data.image} alt={data.title} />
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
</style>
