<script>
  let kuntahaku = '';
  // säähaku-komponentti
  import SRH from './Saahaku.svelte';
  // store johon lämpötilat laitetaan
  import lampotilat from './saaStore.js';
  // storessa olevien lämpötilojen näyttö-komponentti
  import SLIST from './Saalista.svelte';

  // tämä määrää näytetäänkö säähaku komponentti
  $: naytauusi = false;
  // tämä katsoo että kunta on kelvollinen. test komento katsoo onko siinä numeroita, ja sitten käännetään tämä
  $: tarkasta = kuntahaku.length >= 2 && !/\d/.test(kuntahaku);
  // määrää näytetäänkö latausspinneri
  $: lataa = false;

  // kun hae-nappia painetaan poistaa tämä säätiedot näkyvistä (jos sellaisia jo on), näyttää latausspinnerin, odottaa hetken, ja sitten näyttää uudestaan säätiedot, refreshaten ne
  // timeout oma ratkaisuni siihen että saan säätiedot päivittymään. kun pistin false ja true peräkanaan se ei ehtinyt päivittyä, ja päädyin tähän
  function naytaSaa() {
    naytauusi = false;
    lataa = true;
    setTimeout(() => {
      naytauusi = true;
    }, 500);
  }
  let saaid = 0;
  let saamerkinta;
  // kun Saahaku komponentista tulee ladattu dispatch, tämä komponentti pistää id;n, kunnan nimen, huomisen lämpötilan, ja ylihuomisen lämpötilan objektiksi, ja laittaa sen storeen
  // huomisen ja ylihuomisen lämpötilat tulevat dispatch-event:datana
  // latausspinneri myös pois näkyvistä
  function endlataa(event) {
    const unsub = lampotilat.subscribe((saa) => (saamerkinta = saa));
    if (!saamerkinta.find((lmp) => lmp.kunta.toLowerCase() === kuntahaku.toLowerCase()) && saamerkinta.length < 5) {
      lampotilat.update((saamerkinta) => [
        ...saamerkinta,
        { saaid: saaid, kunta: kuntahaku, huomenna: event.detail.huom, ylihuomenna: event.detail.ylihuom },
      ]);
    }
    lataa = false;
    saaid++;
    unsub();
  }
  // jos Saahaku:sta tulee virhedispatch, pistetään pelkästään latausspinneri pois näkyvistä
  function virheEnd() {
    lataa = false;
  }
</script>

<main>
  <div class="saatiedot">
    <h1>lämpötilahaku</h1>
    {#if !naytauusi}
      <h2>Hae kunnan lämpötilatiedot</h2>
    {/if}
    {#if lataa}
      Ladataan
      <!-- spinneri sivulta  https://loading.io/css -->
      <div class="lds-circle"><div /></div>
      <!-- spinner loppuu -->
    {/if}
    {#if !naytauusi}<br /><br /> {/if}

    {#if naytauusi}
      <!-- Saahaku.svelte -->
      <SRH {kuntahaku} on:ladattu={endlataa} on:virhe={virheEnd} />
    {/if}
    <p><input type="text" bind:value={kuntahaku} /></p>
    <p><input type="button" value="hae kuntaa" on:click={naytaSaa} disabled={!tarkasta} /></p>

    {#if !tarkasta}
      <p>anna kelvollinen kunta</p>
    {/if}
    <small>Tiedot:Ilmatieteen laitoksen julkinen data, Tekijä:Jonne Miettinen</small>
    <!-- Saalista.svelte -->
    <SLIST />
  </div>
</main>

<style>
  main {
    background-color: #1cc2f5;
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }
  .saatiedot {
    background-color: white;
    width: 50%;
    margin: auto;
  }
  h1 {
    color: #1cc2f5;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
  /* spinneri otettu  https://loading.io/css sivulta, tässä sen css */
  .lds-circle {
    display: inline-block;
    transform: translateZ(1px);
  }
  .lds-circle > div {
    display: inline-block;
    width: 15px;
    height: 15px;
    margin: 8px;
    border-radius: 50%;
    background: rgb(0, 0, 0);
    animation: lds-circle 2.4s cubic-bezier(0, 0.2, 0.8, 1) infinite;
  }
  @keyframes lds-circle {
    0%,
    100% {
      animation-timing-function: cubic-bezier(0.5, 0, 1, 0.5);
    }
    0% {
      transform: rotateY(0deg);
    }
    50% {
      transform: rotateY(1800deg);
      animation-timing-function: cubic-bezier(0, 0.5, 0.5, 1);
    }
    100% {
      transform: rotateY(3600deg);
    }
  }
  /* spinneri loppuu */
</style>
