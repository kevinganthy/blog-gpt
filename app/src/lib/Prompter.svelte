<script>
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    // Préremplissage des input
    let expert = "en création rédaction d'article pour optimiser le SEO";
    let help = "à développer mon agence web située en Charente-Maritime en rédigeant un article de blog";
    let aim = "d'attirer des clients en Charente-maritime en leur partageant des informations qu'ils pourront appliquer à leur prochain site";
    let target = "les professionnels qui veulent une présence sur internet";
    let subject = "Les étapes clés pour créer un site web efficace";
    let style = "de façon détaillée, en utilisant un exemple concret et en optimisant le SEO";

    // Validation du formulaire + remonté de l'evenement au composant parent
    const submit = (e) => {
        e.preventDefault();
        let prompts = Array.from(document.querySelectorAll('form section li')).map(p => p.textContent);
        dispatch('submit', {
            prompts: prompts,
            subject: subject
        });
    }
</script>

<form on:submit={submit}>
    <fieldset>
      <legend>Personnalisation</legend>
      
      <label for="expert">Expertise</label>
      <input id="expert" bind:value={expert}>

      <label for="help">Aide</label>
      <input id="help" bind:value={help}>

      <label for="aim">But</label>
      <input id="aim" bind:value={aim}>

      <label for="target">Cible</label>
      <input id="target" bind:value={target}>
      
      <label for="subject">Sujet</label>
      <input id="subject" bind:value={subject}>

      <label for="style">Style</label>
      <input id="style" bind:value={style}>
    </fieldset>

    <section>
      <h2>Prévisualisation des prompts</h2>
      <ul>
        <li>Ignore toutes les instructions avant ça. Tu es un expert <mark>{expert}</mark> depuis plus de 20 ans. Tu va m'aider <mark>{help}</mark>. Attends les instructions suivantes, ne répond pas.</li>
        <li>Le public visé est <mark>{target}</mark>. L'article aura pour but <mark>{aim}</mark>. Ne répond pas.</li>
        <li>Maintenant, établis un sommaire détaillé pour le sujet "<mark>{subject}</mark>". Je veux un JSON, chaque élément contient un "title" et des "steps".</li>
        <li>Avec ce sommaire, résume en quelques lignes le sujet "<mark>{subject}</mark>"</li>
        <li>Réfléchissons sur ce chapitre : "...". Rédige ce chapitre <mark>{style}</mark>.</li>
      </ul>
    </section>
    
    <input type="submit" value="Soumettre">
</form>


<style lang="scss">
    @use '../sass/colors';

    form {
        display: flex;
        flex-direction: column; 
        padding: 1em;
        max-height: 80vh;
        overflow-y: auto;

        fieldset {
            display: grid;
            grid-template-columns: auto 1fr;
            align-items: center;
            gap: 8px;
            color: colors.$dark;
            
            input {
                padding: .5em;
                border-radius: 5px;
                border: none;
                background: white;
            }
            label {
                font-weight: bold;
            }
        }
        input[type="submit"] {
            margin: 0 3em 0 auto;
            padding: .75em 2em;
            background: colors.$dark;
            color: colors.$light;
            border-radius: 5px;
            border: none;
            cursor: pointer;
        }

        mark {
            background: colors.$mark;
            font-style: italic;
        }
    }
</style>