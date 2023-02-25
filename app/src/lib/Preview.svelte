<script>
    export let rawData = {};
    let tab = "preview";

    // Comptage des mots de l'article
    const countWords = (data) => {
        if ( data.sections && data.sections.length ) {
            let text = data.sections.map(s => s.title).join(' ')
            + data.sections.map(s => s.content.join(' ')).join(' ')
            return text.split(' ').length;
        }
        else return "--";
    }

    // Récupère le contenu du chapitre en cours
    const filterContent = (title) => {
        try {
            let section = rawData.sections.filter(x => x.title == title);
            return section.length > 0 ? section[0].content : [];
        } catch (error) {
            return [];            
        }
    }
</script>

<section>
    <header>
        <div>
            <input type="radio" name="tab" id="json" checked bind:group={tab} value="json">
            <label for="json">JSON</label>
        </div>
        <div>
            <input type="radio" name="tab" id="preview" bind:group={tab} value="preview">
            <label for="preview">Prévisualisation</label>
        </div>
        <span>{countWords(rawData)} mots</span>
    </header>
    
    <article>
        {#if tab === "json" }
            <pre>{JSON.stringify(rawData, null, 2)}</pre>
        {:else if tab === "preview" }
            <h2>{rawData.title}</h2>
            <p>{rawData.summary ?? ''}</p>

            {#if rawData.synopsis }
                {#each rawData.synopsis.steps ?? rawData.synopsis as chapter }
                    <h3>{chapter.title}</h3>
                    {#if chapter.steps}
                        {#each chapter.steps as step}
                            <h4>{step.title ?? step}</h4>
                            {#each filterContent(step.title ?? step) as content}
                                <p>{content}</p>
                            {/each}
                        {/each}
                    {:else}
                        {#each filterContent(chapter.title ?? chapter) as content}
                            <p>{content}</p>
                        {/each}
                    {/if}
                {/each}
            {/if}
        {/if}
    </article>

</section>

<style lang="scss">
	@use '../sass/colors';

    section {
        max-height: 80vh;
        flex-grow: 1;
        display: flex;
        flex-direction: column;
        
        header {
            display: flex;
            padding: .5em;
            background: lighten(colors.$dark, 60);

            > div {
                display: flex;

                input {
                    display: none;
                }
                input:checked ~ label {
                    background: colors.$dark;
                    color: colors.$light;
                }
                label {
                    margin: .25em 1em;
                    color: colors.$dark;
                    background: colors.$light;
                    padding: .5em 1em;
                    border-radius: 10px;
                    text-decoration: none;
                    font-weight: bold;
                    cursor: pointer;
                }
            }
            span {
                margin: auto 1em auto auto;
            }
        }
        article {
            flex-grow: 1;
            overflow-x: hidden;
            padding: .5em;

            pre {
                white-space: pre-wrap;
                padding: 0 1em;
            }

            h2 {
                text-align: center;
                color: colors.$dark;
            }
            h3 {
                padding: 1em 0 0 0;
                color: colors.$dark;
            }
            p {
                padding: 0 2em;
                text-align: justify;
            }
        }
    }
</style>