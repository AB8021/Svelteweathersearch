<script>
  let kunta;
  import { onMount } from 'svelte';
  import { fly } from 'svelte/transition';
  import { createEventDispatcher } from 'svelte';
  import { fix_position } from 'svelte/internal';
  const dispatch = createEventDispatcher();
  let now = new Date();
  now.setDate(now.getDate() + 1 - 1);
  now = now.toISOString().slice(0, 10);

  let oneday = new Date();
  oneday.setDate(oneday.getDate() + 1);
  oneday = oneday.toISOString().slice(0, 10);

  let twodays = new Date();
  twodays.setDate(twodays.getDate() + 1 * 2);
  let klo = twodays.toISOString().slice(10, 13);
  klo = parseInt(klo.slice(1, 3)) - 1;
  klo = 'T' + klo.toString();
  if (parseInt(klo.slice(1, 3)) > 12) {
    klo = 'T12';
  }
  klo = 'T06';
  twodays = twodays.toISOString().slice(0, 10);

  let endday = new Date();
  endday.setDate(endday.getDate() + 1 * 3);
  endday = endday.toISOString().slice(0, 10);
  $: huomenna = 'ladataan';
  $: ylihuomenna = 'ladataan';
  $: virhe = false;

  export let kuntahaku;

  let linkki = `http://opendata.fmi.fi/wfs?service=WFS&version=2.0.0&request=getFeature&storedquery_id=fmi::forecast::hirlam::surface::point::timevaluepair&place=${kuntahaku}&ep=120&endtime=${endday}T00:00:00Z&&parameters=temperature&`;
  //tämä osa koodista pääosin otettu Parsing XML in ES6 tutoriaalista
  onMount(async () => {
    const response = await fetch(linkki);
    //virheenkäsittelijä lisätty itse
    if (!response.ok) {
      dispatch('virhe');
      virhe = true;
    }
    if (!virhe) {
      // virheenkäsittelyn loppu
      kunta = kuntahaku;
      let xmltext = await response.text();

      let responseDoc = new DOMParser().parseFromString(xmltext, 'application/xml');
      let saatulos = responseDoc.documentElement.textContent;
      // tutoriaalikoodin loppu

      // textcontent stringissä paljon välimerkkejä jotka haittaavat lajittelua, korvataan ne pilkulla, sitten muutetaan pilkkujonot yhdeksi pilkuksi
      saatulos = saatulos.replace(/\s/g, ',').replace(/,,+/g, ',');

      // haluttu lämpötila alkaa aina 21 merkkiä päivämäärän indexofin jälkeen, käytetään sitä paikka arvona
      let paikka = saatulos.indexOf(`${now}T14:00:00Z`) + 21;
      paikka = saatulos.indexOf(`${oneday}T12:00:00Z`) + 21;

      // otetaan stringistä lämpötilan alun kohdassa lämpötila irti. otetaan perästä mukaan runsaasti merkkejä, varmistetaan ettei miinukset, tai moninumeroiset lämpötilat sekoita
      huomenna = saatulos.slice(paikka, paikka + 10);
      // kun tiedetään että kaikki erotellaan pilkulla, voidaan vain poistaa kaikki ylimääräinen pilkusta eteenpäin, muutetaan samalla arvo floatiksi selkeyden vuoksi
      huomenna = parseFloat(huomenna.slice(0, huomenna.indexOf(',')));

      paikka = saatulos.indexOf(`${twodays}${klo}:00:00Z`) + 21;
      ylihuomenna = saatulos.slice(paikka, paikka + 10);
      ylihuomenna = parseFloat(ylihuomenna.slice(0, ylihuomenna.indexOf(',')));
      // dispatch ilmoittaa App.sveltelle että ollaan valmis, annetaan sille samalla lämpötila-arvot
      dispatch('ladattu', { huom: huomenna, ylihuom: ylihuomenna });
    }
  });
</script>

<main>
  {#if !virhe}
    <h2>{kunta}</h2>
    <div transition:fly>
      <p>huomenna lämmintä keskipäivän aikaan: {huomenna}C</p>
      <p>ylihuomenna lämmintä tähän aikaan: {ylihuomenna}C</p>
    </div>
  {/if}
  {#if virhe}
    <div transition:fly>Virhe: Antamallasi arvolla ei löytynyt säätietoja. Oletko varma että kirjoitit oikein suomalaisen kunnan</div>
  {/if}
</main>
