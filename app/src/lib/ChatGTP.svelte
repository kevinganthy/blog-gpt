<script>
    export let token;
    export let retest = true;
    let isError = false;
    let working = false;
    let output = [];
    let wrapper;
   
    // Fonction d'envoi de prompt à chat gpt
    export const send = async (prompt) => {
        log("me", prompt);
        working = true;
        try {
            const response = await fetch("https://api.openai.com/v1/completions", {
                method: "POST",
                headers: {
                    "Authorization": "Bearer " + token,
                    "Content-type": "application/json",
                    "Cache-Control": "no-cache"
                }, 
                body: JSON.stringify({
                    "model": "text-davinci-003",
                    "max_tokens": 4097 - prompt.length,
                    "prompt": prompt
                })
            })
            const json = await response.json();
            const result = json.choices[0].text;
            log("gpt", result);
            working = false;
            return result;
        } catch (error) {
            if ( retest && !isError ) {
                isError = true;
                log("gpt", "ERREUR, nouvelle tentative");
                return send(prompt);
            }
            return "Erreur..."
        }
    }

    // Affichage d'une information dans la conversation
    export const log = (who, what) => {
        output = [...output, {
            who: who,
            what: what
        }];
        setTimeout(() => {
            wrapper.scrollTop = wrapper.scrollHeight;
        }, 100);
    }
</script>

<section bind:this="{wrapper}">
    {#each output as o}
        <p class="{o.who}">{o.what}</p>
    {/each}
    {#if working}
        <div class="spinner">
          <div class="spinner-item"></div>
          <div class="spinner-item"></div>
          <div class="spinner-item"></div>
          <div class="spinner-item"></div>
          <div class="spinner-item"></div>
        </div>
    {/if}
</section>

<style lang="scss">
    @use '../sass/colors';

    section {
        scroll-behavior: smooth;
        display: flex;
        flex-direction: column;
        max-height: 80vh;
        max-width: 600px;
        flex-shrink: 0;
        margin: auto 0;
        overflow-y: auto;
        overflow-x: hidden;

        p {
            background-color: lighten(colors.$dark, 50);
            padding: 1em;
            border-radius: 5px;
            margin: .5em 0 .5em auto;
            max-width: 80%;

            &.gpt {
                background: darken(colors.$light, 10);
                margin: .5em auto .5em 0;
            }
        }
    }

    .spinner {
      $duration: 1000ms;
      $size: 50px;

      grid-row: 1 / span 2;
      grid-column: 2;
      margin: auto auto auto 0;
      width: $size;
      min-height: $size;
      height: $size;
      display: flex;
      align-items: center;
      justify-content: space-evenly;

      .spinner-item {
        width: calc($size / 12);
        height: 80%;
        background: colors.$dark;
        animation: spinner1 $duration ease-in-out infinite;

        @keyframes spinner1 {
          50% {
            transform: scaleY(0.25);
          }
        }
      }
    }
</style>