<script lang="ts">
  const MAX_WORK_DAYS = 249;
  const NORMAL_VACATION_DAYS = 25;
  const WORKDAYS_IN_FOUR_WEEKS = 20;

  let kilometersFromHomeToWork = 2.6;

  let monthsWithoutMotorizedTravel = 3;

  let usedVacationDays = NORMAL_VACATION_DAYS;

  let unplannedAbsenceDays = 0;

  let days = {
    unmotorized: 20,
    rail: 0,
    bus: 0,
    car: 0,
  };
  $: motorizedDaysOutOfTwentyInOffice = days.car + days.bus + days.rail;

  $: days.unmotorized =
    WORKDAYS_IN_FOUR_WEEKS - days.rail - days.bus - days.car;

  $: inOfficeRatio = motorizedDaysOutOfTwentyInOffice / WORKDAYS_IN_FOUR_WEEKS;

  $: actualWorkingDays =
    MAX_WORK_DAYS - usedVacationDays - unplannedAbsenceDays;
  $: totalDaysInOffice = Math.round(actualWorkingDays * inOfficeRatio);

  $: kilometersTraveled = totalDaysInOffice * 2 * kilometersFromHomeToWork;

  $: unmotorizedRatio = monthsWithoutMotorizedTravel / 12;

  $: unmotorizedKilometers = kilometersTraveled * unmotorizedRatio;
  $: motorizedKilometers = kilometersTraveled - unmotorizedKilometers;

  $: kilometers = {
    rail:
      motorizedKilometers * (days.rail / motorizedDaysOutOfTwentyInOffice) || 0,
    bus:
      motorizedKilometers * (days.bus / motorizedDaysOutOfTwentyInOffice) || 0,
    car:
      motorizedKilometers * (days.car / motorizedDaysOutOfTwentyInOffice) || 0,
  };
</script>

<div class="background">
  <div class="content">
    <h1>Utslipp til og fra kontoret</h1>
    Denne kalkulatoren gjør det lettere å fylle ut første side av Knowits klimaundersøkelse.
    <h2>Spørsmål</h2>
    <h3>Tid borte fra kontoret</h3>
    <div class="two_column">
      <label for="walk_bike_months">
        <strong>
          Måneder i året der du hadde hjemmekontor eller gikk og syklet til
          kontoret alle dager
        </strong>
      </label>
      <div>
        <span class="slider-display">{monthsWithoutMotorizedTravel}</span>
        <input
          id="walk_bike_months"
          type="range"
          min="0"
          max="12"
          bind:value={monthsWithoutMotorizedTravel}
          step="1"
        />
      </div>

      <label for="used_vacation_days">
        <strong>Antall feriedager du tok ut.</strong>
        5 uker ferie er 25 dager.
      </label>
      <div>
        <input
          id="used_vacation_days"
          type="number"
          bind:value={usedVacationDays}
          step="1"
          min="0"
          max=""
        />
      </div>

      <label for="unplanned_absence_days">
        <strong>Dager i året du var uplanlagt borte.</strong>
        Sykdom eller hjemmekontor på kort varsel.
      </label>
      <div>
        <input
          id="unplanned_absence_days"
          type="number"
          bind:value={unplannedAbsenceDays}
          step="1"
          min="0"
        />
      </div>
    </div>

    <h3>Fordeling i månedene du var innom kontoret</h3>
    <p>
      En måned der du var innom kontoret har ca. 20 arbeidsdager. Hvordan
      fordelte disse seg på ulike motoriserte transportmidler? Vi antar at du
      gikk, syklet eller var på hjemmekontor de øvrige dagene.
    </p>

    <div class="ratio_display">
      {#if days.unmotorized}
        <div class="unmotorized" style="flex-grow: {days.unmotorized}">
          Gå/sykle/hjemmekontor
        </div>
      {/if}
      {#if days.rail}
        <div class="rail" style="flex-grow: {days.rail}">Skinner</div>
      {/if}
      {#if days.bus}
        <div class="bus" style="flex-grow: {days.bus}">Buss</div>
      {/if}
      {#if days.car}
        <div class="car" style="flex-grow: {days.car}">Bil</div>
      {/if}
    </div>

    <div class="two_column">
      <div>
        <span class="dot unmotorized" />
        <span>Gå/sykle/hjemmekontor</span>
      </div>
      <span>{days.unmotorized}</span>
      <div>
        <span class="dot rail" />
        <label for="tram_days">Tog/trikk/Bybane/T-bane</label>
      </div>
      <div>
        <input
          id="tram_days"
          type="number"
          min="0"
          max={WORKDAYS_IN_FOUR_WEEKS - days.bus - days.car}
          bind:value={days.rail}
          step="1"
        />
      </div>

      <div>
        <span class="dot bus" />
        <label for="bus_days">Buss</label>
      </div>
      <div>
        <input
          id="bus_days"
          type="number"
          min="0"
          max={WORKDAYS_IN_FOUR_WEEKS - days.rail - days.car}
          bind:value={days.bus}
          step="1"
        />
      </div>
      <div>
        <span class="dot car" />
        <label for="car_days">Bil</label>
      </div>
      <div>
        <input
          id="car_days"
          type="number"
          min="0"
          max={WORKDAYS_IN_FOUR_WEEKS - days.rail - days.bus}
          bind:value={days.car}
          step="1"
        />
      </div>
    </div>

    <div class="results">
      <h2>Resultater</h2>
      <div class="two_column">
        <label for="home_work_distance">
          <strong>Kilometer fra hjem til jobb.</strong>
          Bruk «gange» mellom de to adressene på Google Maps
        </label>
        <div>
          <input
            id="home_work_distance"
            type="number"
            bind:value={kilometersFromHomeToWork}
            step="0.1"
          />
        </div>
      </div>

      I fjor hadde du totalt {actualWorkingDays} arbeidsdager ut av {MAX_WORK_DAYS}
      mulige. Av disse var {totalDaysInOffice} på kontoret, og medførte {Math.round(
        motorizedKilometers
      )}
      km motorisert reise. Disse fordelte seg som følger:
      <div class="ratio_display">
        {#if kilometers.rail}
          <div class="rail" style="flex-grow: {kilometers.rail}">Skinner</div>
        {/if}
        {#if kilometers.bus}
          <div class="bus" style="flex-grow: {kilometers.bus}">Buss</div>
        {/if}
        {#if kilometers.car}
          <div class="car" style="flex-grow: {kilometers.car}">Bil</div>
        {/if}
      </div>

      <div class="two_column">
        <div>
          <span class="dot rail" />
          Tog/trikk/bybane/T-bane
        </div>
        <div>{Math.round(kilometers.rail)} km</div>
        <div>
          <span class="dot bus" />
          Buss
        </div>
        <div>{Math.round(kilometers.bus)} km</div>
        <div>
          <span class="dot car" />
          Bil (uspesifisert type)
        </div>
        <div>{Math.round(kilometers.car)} km</div>
      </div>
    </div>
    <footer>
      <p>
        Laget med ❤️ i
        <a href="https://svelte.dev/">Svelte</a>
        av
        <a href="https://github.com/Vages">Eirik Vågeskar.</a>
        Du kan
        <a
          href="https://svelte.dev/repl/3fc04af22c8444f4a96393fe68b9ae83?version=3.16.7"
        >
          fikle med koden selv
        </a>
        om du vil. Takk til Birgitte Rishatt for hjelp med fargevalg.
      </p>
      <p>
        Bli med i #chapter-web på Slack (kontakt
        <a href="mailto:eirik.vageskar@knowit.no">Eirik</a>
        om du ikke jobber i Objectnet).
      </p>
    </footer>
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
