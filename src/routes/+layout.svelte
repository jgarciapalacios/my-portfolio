<script>
	import { page } from '$app/stores';
	let pages = [
		{ url: './', title: 'Home' },
		{ url: './contacts', title: 'Contacts' },
		{ url: './projects', title: 'Projects' },
		{ url: './resume', title: 'Resume' },
		{ url: 'https://github.com/jgarciapalacios', title: 'Github' }
	];
	let localStorage = globalThis.localStorage ?? {};
	let colorScheme = localStorage.colorScheme ?? 'light dark';
	let root = globalThis?.document?.documentElement;
	$: root?.style.setProperty('color-scheme', colorScheme);
</script>

<nav>
	{#each pages as p}
		<a
			href={p.url}
			class:current={'.' + $page.route.id === p.url}
			target={p.url.startsWith('http') ? '_blank' : null}
		>
			{p.title}
		</a>
	{/each}
</nav>
<label class="color-scheme">
	Theme:
	<select bind:value={colorScheme}>
		<option value="light dark"> Automatic </option>
		<option value="light"> Light </option>
		<option value="dark"> Dark </option>
	</select>
</label>
<slot />

<style>
	nav {
		--border-color: oklch(50% 10% 200 / 40%);
		display: flex;
		border-bottom: 2px solid var(--border-color);
	}

	nav a {
		flex: 1;
		text-decoration: none;
		color: inherit;
		text-align: center;
		padding: 0 0.5em;
	}

	nav a.current {
		border-bottom: 0.2em solid oklch(50% 10% 200 / 40%);
		padding-bottom: 0.4em;
		color: var(--color-accent);
	}

	nav a:hover {
		color: var(--color-accent);
		padding-bottom: 0.4em;
		border-bottom: 0.2em solid var(--color-accent);
		background-color: oklch(50% 10% 200 / 40%);
	}

	.color-scheme {
		position: absolute;
		top: 1rem;
		right: 1rem;
		display: inline-flex;
		gap: 4px;
	}
</style>
