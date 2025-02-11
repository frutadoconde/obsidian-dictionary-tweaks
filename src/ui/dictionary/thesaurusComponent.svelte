<script lang="ts">
  import t from "src/l10n/helpers";
  import type { Meaning } from "src/integrations/types";
  import { Notice } from "obsidian";
  import { slide } from "svelte/transition";
  import { copy } from "obsidian-community-lib";

  interface Entry {
      partOfSpeech: string;
      synonyms: Set;
      antonyms: Set;
      open: boolean;
  }

  export let meanings: Meaning[];

  addEventListener("dictionary-collapse", (event: CustomEvent) => {
    thesaurus.forEach((entry) => {
      entry.open = event.detail.open as boolean;
    })
  });

  function wordCopy(word: string) {
copy(
      word,
      () =>
        new Notice(
          t('Copied "{{word}}" to clipboard').replace(/{{word}}/g, word)
        ),
      (error) => new Notice(error)
    );
  }

  function groupByPOS(meanings: Meaning[]): Entry[] {
    /* free dictionary API has synonyms and antonyms inside individual Meaning definitions
     as well as at the top level. This helper groups them at the top level by their parts of speech.*/
    let res: Entry[] = [];
    meanings.forEach((meaning) => {
        const m: Entry = {
            open: false,
            partOfSpeech: meaning.partOfSpeech,
            synonyms: new Set(meaning.synonyms),
            antonyms: new Set(meaning.antonyms),
        };
        meaning.definitions.forEach((definition) => {
            definition.synonyms.forEach((synonym) => {
                synonyms.add(synonym);
            });
            definition.antonyms.forEach((antonym) => {
                antonyms.add(antonym)
            });
        });
        if (m.synonyms.size > 0 || m.antonyms.size > 0) {
            res.push(m);
        }
    });
    return res;
  }
  const thesaurus = groupByPOS(meanings);
</script>

<div class="main">
  {#each thesaurus as entry, i}
    <div class="opener" class:open={entry.open} on:click={() => (entry.open = !entry.open)}>
      <div class="tree-item-icon collapse-icon" style="">
        <svg viewBox="0 0 100 100" class="right-triangle" width="8" height="8">
          <path
            fill="currentColor"
            stroke="currentColor"
            d="M94.9,20.8c-1.4-2.5-4.1-4.1-7.1-4.1H12.2c-3,0-5.7,1.6-7.1,4.1c-1.3,2.4-1.2,5.2,0.2,7.6L43.1,88c1.5,2.3,4,3.7,6.9,3.7 s5.4-1.4,6.9-3.7l37.8-59.6C96.1,26,96.2,23.2,94.9,20.8L94.9,20.8z"
          />
        </svg>
      </div>
      {entry.partOfSpeech}
    </div>
    {#if entry.open}
      <div transition:slide|local={{ duration: 150 }}>
        {#if entry.synonyms.size > 0}
          <div class="synonyms">
            <div class="label">{t("Synonyms:")}</div>
            <p>
              {#each [...entry.synonyms] as synonym, j}
                <span class="synonym" on:click={() => wordCopy(synonym)}>{synonym}</span
                >{#if j < entry.synonyms.size - 1}{", "}{/if}
              {/each}
            </p>
          </div>
        {/if}
        {#if entry.antonyms.size > 0}
          <div class="antonyms">
            <div class="label">{t("Antonyms:")}</div>
            <p>
              {#each [...entry.antonyms] as antonym, j}
                <span class="antonym" on:click={() => wordCopy(antonym)}>{antonym}</span
                >{#if j < entry.antonyms.size - 1}{", "}{/if}
              {/each}
            </p>
          </div>
        {/if}
      </div>
    {/if}
  {/each}
</div>

<style lang="scss">
  .opener {
    display: flex;
    .collapse-icon::after {
      content: "\00a0";
    }
    svg {
      transform: rotate(-90deg);
    }
    &.open svg {
      transform: rotate(0);
    }
  }
  .antonym,
  .synonym {
    transition: 100ms;
    &:hover {
      color: var(--interactive-accent);
      border-radius: 2px;
    }
  }

  .main {
    background-color: var(--background-secondary);
    padding-left: 0.6rem;
    padding-right: 0.6rem;
    padding-top: 0.3rem;
    padding-bottom: 0.3rem;
    margin-bottom: 0.3rem;
    border-radius: 0.3rem;

    blockquote {
      font-style: italic;
      margin: 0 0 1rem;
      padding-left: 1rem;
      border-left: 1px solid var(--background-modifier-border);
    }

    :global(.mark) {
      box-shadow: inset 0 -2px var(--text-faint);
    }
  }

  .label {
    font-size: 0.875em;
    font-weight: bold;
  }

  .antonyms,
  .synonyms {
    padding-top: 1rem;
  }

  .antonyms > p,
  .synonyms > p {
    margin-top: 0;
  }

</style>
