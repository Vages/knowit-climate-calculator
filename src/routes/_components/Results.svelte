<script lang="ts">
  export let actualWorkingDays: number;
  export let totalDaysInOffice: number;
  export let kilometersTraveled: number;

  let kilometersFromHomeToWork = 2.6;

  $: kilometersTraveled = totalDaysInOffice * 2 * kilometersFromHomeToWork;

  $: unmotorizedKilometers = kilometersTraveled * unmotorizedRatio;
  $: motorizedKilometers = kilometersTraveled - unmotorizedKilometers;
</script>

<div class="results">
  <h2>Resultater</h2>
  <label
    >Kilometer hjemmefra til jobb: <input
      type="number"
      bind:value={kilometersFromHomeToWork}
    /></label
  >

  I fjor hadde du totalt {actualWorkingDays} arbeidsdager ut av {MAX_WORK_DAYS}
  mulige. Av disse var {totalDaysInOffice} på kontoret, og medførte {Math.round(
    kilometersTraveled
  )}
  km reise. Disse fordelte seg som følger:
  <div class="ratio_display">
    {#if unmotorizedKilometers}
      <div class="unmotorized" style="flex-grow: {unmotorizedKilometers}">
        Gå/sykle
      </div>
    {/if}
    {#if railKilometers}
      <div class="rail" style="flex-grow: {railKilometers}">Skinner</div>
    {/if}
    {#if busKilometers}
      <div class="bus" style="flex-grow: {busKilometers}">Buss</div>
    {/if}
    {#if carKilometers}
      <div class="car" style="flex-grow: {carKilometers}">Bil</div>
    {/if}
  </div>

  <div class="two_column">
    <div>
      <span class="dot unmotorized" />
      Gå/sykle
    </div>
    <div>{Math.round(unmotorizedKilometers)} km</div>
    <div>
      <span class="dot rail" />
      Tog/trikk/bybane/T-bane
    </div>
    <div>{Math.round(railKilometers)} km</div>
    <div>
      <span class="dot bus" />
      Buss
    </div>
    <div>{Math.round(busKilometers)} km</div>
    <div>
      <span class="dot car" />
      Bil (uspesifisert type)
    </div>
    <div>{Math.round(carKilometers)} km</div>
  </div>
</div>

<style>
  :root {
    --theme-green: #1d88e5;
    --theme-purple: #6835d6;
    --theme-black: #4a4f5f;
    --theme-magenta: #ba288c;
    --theme-red: #e04775;
  }

  :global(body) {
    background: var(--theme-black);
    font-family: "Trebuchet MS", Helvetica, sans-serif;
  }

  h1,
  h2 {
    text-align: center;
    /*margin: 1rem;*/
  }

  h1 {
    padding: 1rem;
    border-radius: 5px;
    margin: 0;
  }

  input[type="range"] {
    margin: 0;
  }

  input[type="number"] {
    width: 100%;
  }

  label {
    display: inline;
  }

  .content {
    margin: 40px auto;
    padding: 20px 40px;
    background: #ddd;
    border-radius: 10px;
    color: var(--theme-black);
    box-shadow: 4px 4px 4px rgba(0, 0, 0, 0.25);
    max-width: 600px;
  }

  .two_column {
    display: grid;
    grid-template-columns: repeat(2, auto);
    grid-auto-flow: row;
    grid-gap: 20px;
    margin: 20px 0;
    align-items: center;
  }

  .slider-display {
    display: inline-block;
    width: 2rem;
  }

  .ratio_display {
    margin: 10px 0;
    display: flex;
    text-align: center;
  }

  .ratio_display > * {
    box-shadow: 0 4px 4px rgba(0, 0, 0, 0.25) inset;
    padding: 1rem;
    color: #fff;
  }

  .ratio_display > *:first-child {
    box-shadow: 4px 4px 4px rgba(0, 0, 0, 0.25) inset;
    border-top-left-radius: 8px;
    border-bottom-left-radius: 8px;
  }

  .ratio_display > *:last-child {
    border-top-right-radius: 8px;
    border-bottom-right-radius: 8px;
  }

  .unmotorized {
    background: var(--theme-green);
  }

  .rail {
    background: var(--theme-purple);
  }

  .bus {
    background: var(--theme-magenta);
  }

  .car {
    background: var(--theme-red);
  }

  .dot {
    margin-bottom: -2px;
    margin-right: 0.2rem;
    box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.25) inset;
    display: inline-block;
    width: 1rem;
    height: 1rem;
    content: "";
    border-radius: 50%;
  }

  .results {
    margin: 40px 0;
  }

  footer {
    text-align: center;
  }
</style>
