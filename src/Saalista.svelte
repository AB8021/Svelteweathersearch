<script>
  import lampotilat from './saaStore.js';
  import { scale } from 'svelte/transition';
  import { flip } from 'svelte/animate';

  // funktio jolla kunnan voi poistaa lämpötilavertailusta
  function poista(saaid) {
    let saamerkinta;
    const unsub = lampotilat.subscribe((saa) => (saamerkinta = saa));
    const saapaikka = saamerkinta.map((item) => item.saaid).indexOf(saaid);
    saamerkinta.splice(saapaikka, 1);
    lampotilat.update((saamerkinta) => saamerkinta);
    unsub();
  }
  console.log($lampotilat.length);
</script>

{#if $lampotilat.length < 1}
  <p>ei kuntia vertailussa</p>
{/if}
{#if !$lampotilat.length < 1}
  <p>napsauta kuntaa poistaaksesi sen vertailusta</p>
{/if}
{#each $lampotilat as lamp (lamp.saaid)}
  <p transition:scale animate:flip on:click={poista(lamp.saaid)}>
    kunnassa <b>{lamp.kunta}</b> lämpötila huomenna <b>{lamp.huomenna}C</b> ja ylihuomenna <b>{lamp.ylihuomenna}C</b>
  </p>
{/each}
