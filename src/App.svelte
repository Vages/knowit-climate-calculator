<script>
	import { tweened } from "svelte/motion"
	import { cubicOut } from "svelte/easing"

	const MAX_WORK_DAYS = 249
	const MAX_VACATION_DAYS = 25
	const MAX_DAYS_EVERY_FOUR_WEEKS = 20

	let days_in_office_every_four_weeks = 20

	$: in_office_ratio =
			days_in_office_every_four_weeks / MAX_DAYS_EVERY_FOUR_WEEKS

	let months_without_motorized_vehicles = 3
	$: unmotorized_ratio = months_without_motorized_vehicles / 12
	let kilometers_from_home_to_work = 2.6

	let used_vacation_days = MAX_VACATION_DAYS

	let unplanned_absence_days = 0

	let rail_days = 0
	let bus_days = 0
	let car_days

	const TWEENED_OPTIONS = { easing: cubicOut }

	$: car_days = days_in_office_every_four_weeks - rail_days - bus_days
	let rail_days_tweened = tweened(rail_days, TWEENED_OPTIONS)
	let bus_days_tweened = tweened(bus_days, TWEENED_OPTIONS)
	let car_days_tweened = tweened(car_days, TWEENED_OPTIONS)

	$: $rail_days_tweened = rail_days
	$: $bus_days_tweened = bus_days
	$: $car_days_tweened = car_days

	$: rail_ratio = rail_days / days_in_office_every_four_weeks
	$: bus_ratio = bus_days / days_in_office_every_four_weeks
	$: car_ratio = car_days / days_in_office_every_four_weeks

	$: actual_working_days =
			MAX_WORK_DAYS - used_vacation_days - unplanned_absence_days
	$: total_days_in_office = Math.round(actual_working_days * in_office_ratio)
	$: kilometers_traveled =
			total_days_in_office * 2 * kilometers_from_home_to_work

	let unmotorized_kilometers, rail_kilometers, bus_kilometers, car_kilometers
	$: unmotorized_kilometers = kilometers_traveled * unmotorized_ratio
	$: motorized_kilometers = kilometers_traveled - unmotorized_kilometers

	$: rail_kilometers = motorized_kilometers * rail_ratio
	$: bus_kilometers = motorized_kilometers * bus_ratio
	$: car_kilometers = motorized_kilometers * car_ratio

	let unmotorized_kilometers_tweened = tweened(
			unmotorized_kilometers,
			TWEENED_OPTIONS
	)
	let rail_kilometers_tweened = tweened(rail_kilometers, TWEENED_OPTIONS)
	let bus_kilometers_tweened = tweened(bus_kilometers, TWEENED_OPTIONS)
	let car_kilometers_tweened = tweened(car_kilometers, TWEENED_OPTIONS)

	$: $unmotorized_kilometers_tweened = unmotorized_kilometers
	$: $rail_kilometers_tweened = rail_kilometers
	$: $bus_kilometers_tweened = bus_kilometers
	$: $car_kilometers_tweened = car_kilometers
</script>

<style>
	:root {
		/* Color Theme Swatches in Hex */
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
		margin: 3rem auto;
		padding: 3rem;
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
		grid-gap: 2rem;
		margin: 2rem 0;
		align-items: center;
	}

	.slider-display {
		display: inline-block;
		width: 2rem;
	}

	.ratio_display {
		margin: 1rem 0;
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
		border-top-left-radius: 0.75rem;
		border-bottom-left-radius: 0.75rem;
	}

	.ratio_display > *:last-child {
		border-top-right-radius: 0.75rem;
		border-bottom-right-radius: 0.75rem;
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
		margin-top: 4rem;
	}

	footer {
		text-align: center;
	}
</style>

<div class="background">
	<div class="content">
		<h1>Mellom hjemme og jobb: Kilometerfordeling</h1>
		Denne kalkulatoren gjør det lettere å fylle ut første side av Knowits
		klimaundersøkelse.
		<h2>Spørsmål</h2>
		<h3>Grunnleggende</h3>
		<div class="two_column">
			<label for="home_work_distance">
				Kilometer fra hjem til jobb (bruk «gange» mellom de to adressene på
				Google Maps)
			</label>
			<div>
				<input
						id="home_work_distance"
						type="number"
						bind:value={kilometers_from_home_to_work}
						step="0.1" />
			</div>

			<div>
				<span class="dot unmotorized" />
				<label for="walk_bike_months">
					Måneder i året der du gikk eller syklet alle dager
				</label>
			</div>
			<div>
				<span class="slider-display">{months_without_motorized_vehicles}</span>
				<input
						id="walk_bike_months"
						type="range"
						min="0"
						max="12"
						bind:value={months_without_motorized_vehicles}
						step="1" />
			</div>

			<label for="used_vacation_days">
				Antall feriedager du tok ut (5 uker ferie er 25 dager)
			</label>
			<div>
				<input
						id="used_vacation_days"
						type="number"
						bind:value={used_vacation_days}
						step="1"
						min="0"
						max="" />
			</div>

			<label for="unplanned_absence_days">
				Dager i året du var uforutsett syk eller tok hjemmekontor på kort
				varsel.
			</label>
			<div>
				<input
						id="unplanned_absence_days"
						type="number"
						bind:value={unplanned_absence_days}
						step="1"
						min="0" />
			</div>

			<label for="days_in_office_every_four_weeks">
				<strong>Hvor ofte er du på kontoret:</strong>
				Ut av 20 arbeidsdager (4 uker), hvor mange av dem planlegger du å være
				på kontoret (kontra hjemmekontor)?
			</label>
			<div>
				<input
						id="days_in_office_every_four_weeks"
						type="number"
						bind:value={days_in_office_every_four_weeks}
						step="1"
						min="0"
						max="20" />
			</div>
		</div>

		<h3>Fordeling av kontordager</h3>
		<p>
			Du oppgir at du planlegger å bruke {days_in_office_every_four_weeks} av {MAX_DAYS_EVERY_FOUR_WEEKS}
			arbeidsdager på kontoret innenfor 4 uker. Gitt at du ikke må ta ut noe
			uplanlagt fravær, hvordan fordeler dagene seg på ulike reisemidler?
		</p>
		<p>
			<em>
				Tips 1: Hvis du ikke bytter reisemiddel fra dag til dag, men bytter
				reisemiddel underveis i reisen, kan du heller bruke sliderne til å
				justere forholdet mellom reisemidlene.
			</em>
		</p>
		<p>
			<em>
				Tips 2: Hvis du går deler av strekningen, bare juster ned «Kilometer fra
				hjem til jobb» tilsvarende gåavstanden; gange har et neglisjerbart
				klimagassutslipp.
			</em>
		</p>

		<div class="ratio_display">
			{#if $rail_days_tweened}
				<div class="rail" style="flex-grow: {$rail_days_tweened}">Skinner</div>
			{/if}
			{#if $bus_days_tweened}
				<div class="bus" style="flex-grow: {$bus_days_tweened}">Buss</div>
			{/if}
			{#if $car_days_tweened}
				<div class="car" style="flex-grow: {$car_days_tweened}">Bil</div>
			{/if}
		</div>

		<div class="two_column">
			<div>
				<span class="dot rail" />
				<label for="tram_days">Tog/trikk/Bybane/T-bane</label>
			</div>
			<div>
				<span class="slider-display">{rail_days}</span>
				<input
						id="tram_days"
						type="range"
						min="0"
						max={days_in_office_every_four_weeks - bus_days}
						bind:value={rail_days}
						step="1" />
			</div>

			<div>
				<span class="dot bus" />
				<label for="bus_days">Buss</label>
			</div>
			<div>

				<span class="slider-display">{bus_days}</span>
				<input
						id="bus_days"
						type="range"
						min="0"
						max={days_in_office_every_four_weeks - rail_days}
						bind:value={bus_days}
						step="1" />
			</div>
			<div>
				<span class="dot car" />
				<span>Bildager</span>
			</div>
			<span>{car_days}</span>
		</div>

		<div class="results">
			<h2>Resultater</h2>
			I fjor hadde du totalt {actual_working_days} arbeidsdager ut av {MAX_WORK_DAYS}
			mulige. Av disse var {total_days_in_office} på kontoret, og medførte {Math.round(kilometers_traveled)}
			km reise. Disse fordelte seg som følger:
			<div class="ratio_display">
				{#if $unmotorized_kilometers_tweened}
					<div
							class="unmotorized"
							style="flex-grow: {$unmotorized_kilometers_tweened}">
						Gå/sykle
					</div>
				{/if}
				{#if $rail_kilometers_tweened}
					<div class="rail" style="flex-grow: {$rail_kilometers_tweened}">
						Skinner
					</div>
				{/if}
				{#if $bus_kilometers_tweened}
					<div class="bus" style="flex-grow: {$bus_kilometers_tweened}">
						Buss
					</div>
				{/if}
				{#if $car_kilometers_tweened}
					<div class="car" style="flex-grow: {$car_kilometers_tweened}">
						Bil
					</div>
				{/if}
			</div>

			<div class="two_column">
				<div>
					<span class="dot unmotorized" />
					Gå/sykle
				</div>
				<div>{Math.round(unmotorized_kilometers)} km</div>
				<div>
					<span class="dot rail" />
					Tog/trikk/bybane/T-bane
				</div>
				<div>{Math.round(rail_kilometers)} km</div>
				<div>
					<span class="dot bus" />
					Buss
				</div>
				<div>{Math.round(bus_kilometers)} km</div>
				<div>
					<span class="dot car" />
					Bil (uspesifisert type)
				</div>
				<div>{Math.round(car_kilometers)} km</div>
			</div>
		</div>
		<footer>
			Lagd med ❤️ i
			<a href="https://svelte.dev/">Svelte</a>
			av
			<a href="https://github.com/Vages">Eirik Vågeskar.</a>
			Du kan
			<a
					href="https://svelte.dev/repl/3fc04af22c8444f4a96393fe68b9ae83?version=3.16.7">
				sjekke ut koden selv,
			</a>
			om du vil. Takk til Birgitte Rishatt for hjelp med fargevalg.
		</footer>
	</div>
</div>
