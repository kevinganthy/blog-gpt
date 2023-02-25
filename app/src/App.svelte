<script>
	import ChatGtp from "./lib/ChatGTP.svelte";
    import Preview from "./lib/Preview.svelte";
	import Prompter from './lib/Prompter.svelte';
	
	// Instance du composant ChatGpt pour accéder à ses méthodes
	let compGpt;
	
	let rawData = {};
	import tmp from './assets/rawdata.json';
	rawData = tmp;
	let working = false;

	// Utilisation du localstorage pour stocker le token de l'api
	let token = localStorage.getItem('gpt-token');
	if ( !token ) {
		token = window.prompt("Token GPT", "");
		localStorage.setItem('gpt-token', token);
	}
	
	
	// Récupération du submit du composant Prompter
	const onSubmitPrompt = (event) => {
		working = true;
		rawData = {};
		rawData.title = event.detail.subject;
		start(event.detail.prompts);		
	}

	// Lancement du process principal
	const start = async (prompts) => {
		// Récupère le dernier prompt qui servira plusieurs fois
		const lastPrompt = prompts.pop();

		for (const index in prompts) {
			const i = parseInt(index)
			// Envoi une  requete à chat gpt
			let response = await compGpt.send(prompts[i]);

			// En fonction u prompt
			switch (i) {
				// création du sommaire
				case prompts.length-2:
					response = parseSynopsis(response);
					// Tente une deuxième fois
					if ( response === false ) {
						response = await compGpt.send(prompts[i]);
						response = parseSynopsis(response);
					};
					if ( response === false ) return false;
					rawData.synopsis = response;
					break;
				// création du résumé
				case prompts.length-1:
					rawData.summary = response
					rawData.sections = [];
					// Démarre l'écriture des chapitres
					await throughSynopsis(lastPrompt);
					break;
			}
		}

		working = false;
	}

	// Tente de parser les sommaire en JSON
	const parseSynopsis = (response) => {
		try {
			response = response.replace(/`/g, '');
			response = response.replace(/json/g, '');
			return JSON.parse(response);
		} catch (error) {         
			compGpt.log("GPT", "Erreur de parsing du sommaire");
			return false;
		}
	}

	// Parcours le sommaire pour rédiger chaque chapitre
	const throughSynopsis = async (lastPrompt) => {
		// Gère différents formats de sommaire
		let steps = rawData.synopsis.steps ?? rawData.synopsis;
		for (const item of steps) {
			if ( item.steps ) {
				for (const subitem of item.steps) {
					let title = subitem.title ?? subitem;
					await write(lastPrompt, title)
				}
			}
			else await write(lastPrompt, item.title)
		}
	}

	// Envoi la requêtre d'écriture pour un chapitre
	const write = async (basePrompt, title) => {
		let prompt = basePrompt.replace(`"..."`, title);
		let response = await compGpt.send(prompt);
		
		rawData.sections = [...rawData.sections, {
			title: title,
			content: response.split('\n\n').filter(x => x)
		}]
	}
</script>


<header>
	<h1>Blog GPT</h1>
</header>

<main>
	<ChatGtp {token} bind:this={compGpt}/>
	{#if working === false}
		<Prompter on:submit={onSubmitPrompt}/>
	{:else}
		<Preview {rawData} />
	{/if}
</main>



<style lang="scss">
	@use './sass/colors';

	header {
		background: colors.$dark;
		color: colors.$light;
		padding: .5em 1em;
	}

	main {
		flex-grow: 1;
		display: flex;
		padding: 1em;
	}
</style>