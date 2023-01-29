<script>
	const firstCharacters = {
    Avicii: 'A',
    Kygo: 'K',
    'Jonas Blue': 'JB'
	}

	// the name is updated whenever the prop value changes...
	export let name;

	// ...but the "firstCharacter" variable is fixed upon initialisation of the component
	const firstCharacter = firstCharacters[name];
</script>

<p>
	<span>The first character for { name } is { firstCharacter }</span>
</p>

<style>
	p {
		margin: 0.8em 0;
	}
	span {
		display: inline-block;
		padding: 0.2em 1em 0.3em;
		text-align: center;
		border-radius: 0.2em;
		background-color: #FFDFD3;
	}
</style>
