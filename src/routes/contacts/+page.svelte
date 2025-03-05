<svelte:head>
	<title>Contact me</title>
</svelte:head>

<div id="contact-page">
	<h1>Contact Me</h1>
	<form action="mailto:javs111103@gmail.com" method="GET">
		<label>
			Email:
			<input name="email" type="email" required />
		</label>
		<br />
		<label>
			Subject:
			<input name="subject" />
		</label>
		<br />
		<label>
			Message:
			<textarea name="body" rows="4" />
		</label>
		<br />
		<button>Send</button>
	</form>
</div>
