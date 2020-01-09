<script>
	import { tweened } from "svelte/motion"
	import { cubicOut } from "svelte/easing"

	const MAX_WORK_DAYS = 249
	const MAX_VACATION_DAYS = 25
	const MAX_DAYS_EVERY_FOUR_WEEKS = 20

	let kilometers_from_home_to_work = 2.6

	let months_without_motorized_vehicles = 3
	$: unmotorized_ratio = months_without_motorized_vehicles / 12

	let used_vacation_days = MAX_VACATION_DAYS

	let planned_home_office_days = 0

	$: days_in_office_every_four_weeks =
		MAX_DAYS_EVERY_FOUR_WEEKS - planned_home_office_days

	$: in_office_ratio =
		days_in_office_every_four_weeks / MAX_DAYS_EVERY_FOUR_WEEKS

	let unplanned_absence_days = 0

	let rail_days = 0
	let bus_days = 0

	const TWEENED_OPTIONS = { easing: cubicOut }

	$: car_days = days_in_office_every_four_weeks - rail_days - bus_days
	let rail_days_tweened = tweened(0, TWEENED_OPTIONS)
	let bus_days_tweened = tweened(0, TWEENED_OPTIONS)
	let car_days_tweened = tweened(
		1, // Avoid blinking on page load
		TWEENED_OPTIONS
	)

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

	$: (() => {
		// Prevent a negative amount of days if user adjusts number of days in home office
		if (rail_days + bus_days > days_in_office_every_four_weeks) {
			if (rail_days) {
				rail_days = days_in_office_every_four_weeks - bus_days
			}
			if (bus_days) {
				bus_days = days_in_office_every_four_weeks - rail_days
			}
		}
	})()
</script>

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

<div class="background">
	<div class="content">
		<h1>Utslipp til og fra kontoret</h1>
		Denne kalkulatoren gjør det lettere å fylle ut første side av Knowits
		klimaundersøkelse.
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
					bind:value={kilometers_from_home_to_work}
					step="0.1" />
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
				<strong>Antall feriedager du tok ut.</strong>
				5 uker ferie er 25 dager.
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

			<label for="days_in_office_every_four_weeks">
				<strong>Hvor ofte har du planlagt hjemmekontor?</strong>
				Ut av 20 arbeidsdager (4 uker), hvor mange av dem har du regelmessig
				hjemmekontor?
			</label>
			<div>
				<input
					id="days_in_office_every_four_weeks"
					type="number"
					bind:value={planned_home_office_days}
					step="1"
					min="0"
					max="20" />
			</div>

			<label for="unplanned_absence_days">
				<strong>Dager i året du var uplanlagt borte.</strong>
				Sykdom eller hjemmekontor på kort varsel.
			</label>
			<div>
				<input
					id="unplanned_absence_days"
					type="number"
					bind:value={unplanned_absence_days}
					step="1"
					min="0" />
			</div>

		</div>

		<h3>Fordeling av kontordager</h3>
		<p>
			Fordi du oppgir at du gjennomsnittlig bruker {planned_home_office_days} av
			{MAX_DAYS_EVERY_FOUR_WEEKS} dager på hjemmekontor, regner vi med du bruker
			{days_in_office_every_four_weeks} av {MAX_DAYS_EVERY_FOUR_WEEKS} dager på
			kontoret.
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
			<p>
				Lagd med ❤️ i
				<a href="https://svelte.dev/">Svelte</a>
				av
				<a href="https://github.com/Vages">Eirik Vågeskar.</a>
				Du kan
				<a
					href="https://svelte.dev/repl/3fc04af22c8444f4a96393fe68b9ae83?version=3.16.7">
					fikle med koden selv
				</a>
				om du vil. Takk til Birgitte Rishatt for hjelp med fargevalg.
			</p>
			<p>
				Bli med i #guild-svelte på Slack (kontakt
				<a href="mailto:eirik.vageskar@knowit.no">Eirik</a>
				om du ikke jobber i Objectnet).
			</p>
		</footer>
	</div>
</div>
