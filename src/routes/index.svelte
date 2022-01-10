<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";

  const MAX_WORK_DAYS = 249;
  const MAX_VACATION_DAYS = 25;
  const MAX_DAYS_EVERY_FOUR_WEEKS = 20;

  let kilometersFromHomeToWork = 2.6;

  let monthsWithoutMotorizedVehicles = 3;
  $: unmotorizedRatio = monthsWithoutMotorizedVehicles / 12;

  let usedVacationDays = MAX_VACATION_DAYS;

  let plannedHomeOfficeDays = 0;

  $: daysInOfficeEveryFourWeeks =
    MAX_DAYS_EVERY_FOUR_WEEKS - plannedHomeOfficeDays;

  $: inOfficeRatio =
    daysInOfficeEveryFourWeeks / MAX_DAYS_EVERY_FOUR_WEEKS;

  let unplannedAbsenceDays = 0;

  let railDays = 0;
  let busDays = 0;

  const TWEENED_OPTIONS = { easing: cubicOut };

  $: carDays = daysInOfficeEveryFourWeeks - railDays - busDays;
  let railDaysTweened = tweened(0, TWEENED_OPTIONS);
  let busDaysTweened = tweened(0, TWEENED_OPTIONS);
  let carDaysTweened = tweened(
    1, // Avoid blinking on page load
    TWEENED_OPTIONS
  );

  $: $railDaysTweened = railDays;
  $: $busDaysTweened = busDays;
  $: $carDaysTweened = carDays;

  $: railRatio = railDays / daysInOfficeEveryFourWeeks;
  $: busRatio = busDays / daysInOfficeEveryFourWeeks;
  $: carRatio = carDays / daysInOfficeEveryFourWeeks;

  $: actualWorkingDays =
    MAX_WORK_DAYS - usedVacationDays - unplannedAbsenceDays;
  $: totalDaysInOffice = Math.round(actualWorkingDays * inOfficeRatio);
  $: kilometersTraveled =
    totalDaysInOffice * 2 * kilometersFromHomeToWork;

  $: unmotorizedKilometers = kilometersTraveled * unmotorizedRatio;
  $: motorizedKilometers = kilometersTraveled - unmotorizedKilometers;

  $: railKilometers = motorizedKilometers * railRatio;
  $: busKilometers = motorizedKilometers * busRatio;
  $: carKilometers = motorizedKilometers * carRatio;

  let unmotorizedKilometersTweened = tweened(
    unmotorizedKilometers,
    TWEENED_OPTIONS
  );
  let railKilometersTweened = tweened(railKilometers, TWEENED_OPTIONS);
  let busKilometersTweened = tweened(busKilometers, TWEENED_OPTIONS);
  let carKilometersTweened = tweened(carKilometers, TWEENED_OPTIONS);

  $: $unmotorizedKilometersTweened = unmotorizedKilometers;
  $: $railKilometersTweened = railKilometers;
  $: $busKilometersTweened = busKilometers;
  $: $carKilometersTweened = carKilometers;

  $: (() => {
    // Prevent a negative amount of days if user adjusts number of days in home office
    if (railDays + busDays > daysInOfficeEveryFourWeeks) {
      if (railDays) {
        railDays = daysInOfficeEveryFourWeeks - busDays;
      }
      if (busDays) {
        busDays = daysInOfficeEveryFourWeeks - railDays;
      }
    }
  })();
</script>

<div class="background">
  <div class="content">
    <h1>Utslipp til og fra kontoret</h1>
    Denne kalkulatoren gjør det lettere å fylle ut første side av Knowits klimaundersøkelse.
    <h2>Spørsmål</h2>
    <h3>Grunnleggende</h3>
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

      <div>
        <span class="dot unmotorized" />
        <label for="walk_bike_months">
          <strong>
            Måneder i året der du gikk eller syklet
            <em>alle</em>
            arbeidsdager.
          </strong>
        </label>
      </div>
      <div>
        <span class="slider-display">{monthsWithoutMotorizedVehicles}</span>
        <input
          id="walk_bike_months"
          type="range"
          min="0"
          max="12"
          bind:value={monthsWithoutMotorizedVehicles}
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

      <label for="days_in_office_every_four_weeks">
        <strong>Hvor ofte har du planlagt hjemmekontor?</strong>
        Ut av 20 arbeidsdager (4 uker), hvor mange av dem har du regelmessig hjemmekontor?
      </label>
      <div>
        <input
          id="days_in_office_every_four_weeks"
          type="number"
          bind:value={plannedHomeOfficeDays}
          step="1"
          min="0"
          max="20"
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

    <h3>Fordeling av kontordager</h3>
    <p>
      Fordi du oppgir at du gjennomsnittlig bruker {plannedHomeOfficeDays} av
      {MAX_DAYS_EVERY_FOUR_WEEKS} dager på hjemmekontor, regner vi med du bruker
      {daysInOfficeEveryFourWeeks} av {MAX_DAYS_EVERY_FOUR_WEEKS} dager på kontoret.
      <strong>
        Gitt at du ikke må ta ut noe uplanlagt fravær og ikke går/sykler til
        jobb, hvordan fordeler dagene dine seg på ulike reisemidler?
      </strong>
    </p>
    <p>
      <em>
        Tips 1: Hvis du ikke bytter reisemiddel fra dag til dag, men bytter
        reisemiddel underveis i reisen, kan du heller bruke skyveknappene til å
        justere forholdet mellom reisemidlene.
      </em>
    </p>
    <p>
      <em>
        Tips 2: Hvis du går deler av strekningen og ikke vil ta med gåavstanden
        i beregningen, juster ned «Kilometer fra hjem til jobb» tilsvarende
        gåavstanden (fordi gange har et neglisjerbart klimaavtrykk).
      </em>
    </p>

    <div class="ratio_display">
      {#if $railDaysTweened}
        <div class="rail" style="flex-grow: {$railDaysTweened}">Skinner</div>
      {/if}
      {#if $busDaysTweened}
        <div class="bus" style="flex-grow: {$busDaysTweened}">Buss</div>
      {/if}
      {#if $carDaysTweened}
        <div class="car" style="flex-grow: {$carDaysTweened}">Bil</div>
      {/if}
    </div>

    <div class="two_column">
      <div>
        <span class="dot rail" />
        <label for="tram_days">Tog/trikk/Bybane/T-bane</label>
      </div>
      <div>
        <span class="slider-display">{railDays}</span>
        <input
          id="tram_days"
          type="range"
          min="0"
          max={daysInOfficeEveryFourWeeks - busDays}
          bind:value={railDays}
          step="1"
        />
      </div>

      <div>
        <span class="dot bus" />
        <label for="bus_days">Buss</label>
      </div>
      <div>
        <span class="slider-display">{busDays}</span>
        <input
          id="bus_days"
          type="range"
          min="0"
          max={daysInOfficeEveryFourWeeks - railDays}
          bind:value={busDays}
          step="1"
        />
      </div>
      <div>
        <span class="dot car" />
        <span>Bildager</span>
      </div>
      <span>{carDays}</span>
    </div>

    <div class="results">
      <h2>Resultater</h2>
      I fjor hadde du totalt {actualWorkingDays} arbeidsdager ut av {MAX_WORK_DAYS}
      mulige. Av disse var {totalDaysInOffice} på kontoret, og medførte {Math.round(
        kilometersTraveled
      )}
      km reise. Disse fordelte seg som følger:
      <div class="ratio_display">
        {#if $unmotorizedKilometersTweened}
          <div
            class="unmotorized"
            style="flex-grow: {$unmotorizedKilometersTweened}"
          >
            Gå/sykle
          </div>
        {/if}
        {#if $railKilometersTweened}
          <div class="rail" style="flex-grow: {$railKilometersTweened}">
            Skinner
          </div>
        {/if}
        {#if $busKilometersTweened}
          <div class="bus" style="flex-grow: {$busKilometersTweened}">
            Buss
          </div>
        {/if}
        {#if $carKilometersTweened}
          <div class="car" style="flex-grow: {$carKilometersTweened}">
            Bil
          </div>
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
