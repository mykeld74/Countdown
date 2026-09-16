<script>
	import { onMount } from 'svelte';

	/** Oct 14 2026, 4pm Mountain Daylight Time */
	const endTime = new Date('2026-10-14T16:00:00-06:00').getTime();
	const breckLat = 39.4817;
	const breckLon = -106.0384;

	const weatherLabels = /** @type {Record<number, string>} */ ({
		0: 'Clear skies',
		1: 'Mostly clear',
		2: 'Partly cloudy',
		3: 'Overcast',
		45: 'Foggy',
		48: 'Icy fog',
		51: 'Light drizzle',
		53: 'Drizzle',
		55: 'Heavy drizzle',
		61: 'Light rain',
		63: 'Rain',
		65: 'Heavy rain',
		71: 'Light snow',
		73: 'Snow',
		75: 'Heavy snow',
		77: 'Snow grains',
		80: 'Light showers',
		81: 'Showers',
		82: 'Heavy showers',
		85: 'Snow showers',
		86: 'Heavy snow showers',
		95: 'Thunderstorm',
		96: 'Thunder + hail',
		99: 'Heavy hail'
	});

	const getParts = () => {
		const remaining = Math.floor((endTime - Date.now()) / 1000);
		if (remaining <= 0) {
			return { arrived: true, daysLeft: 0, hoursLeft: 0, minutesLeft: 0, secondsLeft: 0 };
		}
		return {
			arrived: false,
			daysLeft: Math.floor(remaining / 86400),
			hoursLeft: Math.floor((remaining / 3600) % 24),
			minutesLeft: Math.floor((remaining / 60) % 60),
			secondsLeft: remaining % 60
		};
	};

	let parts = $state({
		arrived: false,
		daysLeft: 0,
		hoursLeft: 0,
		minutesLeft: 0,
		secondsLeft: 0
	});
	let ready = $state(false);
	let mountainTime = $state('');
	let tempF = $state(/** @type {number | null} */ (null));
	let conditionLabel = $state('');

	const pad = (n) => String(n).padStart(2, '0');

	const tick = () => {
		parts = getParts();
	};

	const updateMountainClock = () => {
		mountainTime = new Intl.DateTimeFormat('en-US', {
			timeZone: 'America/Denver',
			hour: 'numeric',
			minute: '2-digit',
			hour12: true
		}).format(new Date());
	};

	const fetchConditions = async () => {
		try {
			const url = new URL('https://api.open-meteo.com/v1/forecast');
			url.searchParams.set('latitude', String(breckLat));
			url.searchParams.set('longitude', String(breckLon));
			url.searchParams.set('current', 'temperature_2m,weather_code');
			url.searchParams.set('temperature_unit', 'fahrenheit');
			url.searchParams.set('timezone', 'America/Denver');

			const res = await fetch(url);
			if (!res.ok) return;
			const data = await res.json();
			tempF = Math.round(data.current.temperature_2m);
			conditionLabel = weatherLabels[data.current.weather_code] ?? 'Mountain weather';
		} catch {
			/* keep last known / empty */
		}
	};

	const leafCount = $derived(
		!ready
			? 8
			: parts.arrived
				? 28
				: parts.daysLeft <= 3
					? 22
					: parts.daysLeft <= 14
						? 16
						: parts.daysLeft <= 30
							? 11
							: 7
	);

	const leaves = $derived.by(() => {
		const palette = ['#f0c14b', '#e8a84a', '#d4843a', '#f5d76e', '#c96b2d'];
		return Array.from({ length: leafCount }, (_, id) => {
			const rand = (n) => {
				const x = Math.sin(id * 9973 + leafCount * 13 + n * 17) * 10000;
				return x - Math.floor(x);
			};
			return {
				id,
				left: rand(1) * 100,
				delay: rand(2) * -20,
				duration: 11 + rand(3) * 16,
				size: 10 + rand(4) * 16,
				drift: -40 + rand(5) * 80,
				spin: 120 + rand(6) * 280,
				tone: palette[Math.floor(rand(7) * palette.length)],
				sway: 8 + rand(8) * 18
			};
		});
	});

	onMount(() => {
		tick();
		ready = true;
		updateMountainClock();
		fetchConditions();

		const countdownId = setInterval(tick, 250);
		const clockId = setInterval(updateMountainClock, 15_000);
		const weatherId = setInterval(fetchConditions, 15 * 60_000);

		return () => {
			clearInterval(countdownId);
			clearInterval(clockId);
			clearInterval(weatherId);
		};
	});
</script>

<svelte:head>
	<title>Breckenridge</title>
</svelte:head>

<main class="scene" class:arrived={parts.arrived}>
	<div class="sky" aria-hidden="true"></div>
	<div class="sunGlow" aria-hidden="true"></div>

	<div class="aspens" aria-hidden="true">
		{#each leaves as leaf (leaf.id)}
			<span
				class="leaf"
				style="
					--left: {leaf.left}%;
					--delay: {leaf.delay}s;
					--duration: {leaf.duration}s;
					--size: {leaf.size}px;
					--drift: {leaf.drift}px;
					--spin: {leaf.spin}deg;
					--tone: {leaf.tone};
					--sway: {leaf.sway}px;
				"
			></span>
		{/each}
	</div>

	<svg class="ridges" viewBox="0 0 1200 400" preserveAspectRatio="none" aria-hidden="true">
		<!-- distant: sparse, taller massifs -->
		<path
			class="ridge far"
			d="M0 400 L0 248
			   L95 205 L145 228 L250 120 L310 175 L385 155
			   L480 78 L545 148 L620 118 L710 168
			   L820 72 L890 142 L970 108 L1055 162
			   L1140 125 L1200 158 L1200 400 Z"
		/>
		<!-- middle: denser, offset rhythm -->
		<path
			class="ridge mid"
			d="M0 400 L0 292
			   L60 262 L140 285 L230 210 L290 255 L370 188
			   L430 240 L520 165 L590 225 L680 178
			   L745 238 L840 155 L915 220 L1005 185
			   L1085 245 L1200 198 L1200 400 Z"
		/>
		<!-- near: low rolling foothills -->
		<path
			class="ridge near"
			d="M0 400 L0 338
			   L110 312 L200 335 L320 298 L430 328
			   L540 288 L650 322 L760 295 L870 325
			   L980 300 L1090 320 L1200 308 L1200 400 Z"
		/>
	</svg>

	<section class="content">
		<p class="eyebrow">Countdown to</p>
		<h1 class="brand">Breckenridge</h1>
		<p class="lede">
			{#if parts.arrived}
				Aspens gold. High country ahead.
			{:else}
				Until we leave.
			{/if}
		</p>

		{#if ready && !parts.arrived}
			<div class="clock" aria-live="polite">
				<div class="unit">
					<span class="value">{parts.daysLeft}</span>
					<span class="label">Days</span>
				</div>
				<span class="sep" aria-hidden="true">:</span>
				<div class="unit">
					<span class="value">{pad(parts.hoursLeft)}</span>
					<span class="label">Hours</span>
				</div>
				<span class="sep" aria-hidden="true">:</span>
				<div class="unit">
					<span class="value">{pad(parts.minutesLeft)}</span>
					<span class="label">Minutes</span>
				</div>
				<span class="sep" aria-hidden="true">:</span>
				<div class="unit">
					<span class="value">{pad(parts.secondsLeft)}</span>
					<span class="label">Seconds</span>
				</div>
			</div>
		{:else if ready && parts.arrived}
			<p class="arrivedLine">We're Outta Here!</p>
		{/if}

		{#if ready}
			<p class="pulse">
				<span>Mountain time {mountainTime || '—'}</span>
				{#if tempF !== null}
					<span class="dot" aria-hidden="true">·</span>
					<span>{tempF}°F · {conditionLabel}</span>
				{/if}
			</p>
		{/if}
	</section>
</main>

<style>
	:global(html, body) {
		margin: 0;
		height: 100%;
	}

	:global(body) {
		font-family: 'Sora', sans-serif;
		color: var(--ink);
		background: #b9c9d8;
	}

	.scene {
		--ink: #1a2a24;
		--ink-soft: color-mix(in oklab, var(--ink) 70%, white);
		--glow: #f0b27a;
		--aspen: #e8b84a;

		position: relative;
		isolation: isolate;
		overflow: hidden;
		box-sizing: border-box;
		min-height: 100vh;
		min-height: 100dvh;
		display: grid;
		place-items: center;
		padding: clamp(1.5rem, 4vw, 3rem);
	}

	.sky {
		position: absolute;
		inset: 0;
		z-index: -3;
		background:
			radial-gradient(
				ellipse 70% 42% at 50% 100%,
				color-mix(in oklab, var(--aspen) 28%, transparent),
				transparent 58%
			),
			radial-gradient(
				ellipse 80% 50% at 50% 108%,
				color-mix(in oklab, var(--glow) 50%, transparent),
				transparent 55%
			),
			linear-gradient(180deg, #8eabc4 0%, #b7c9dc 36%, #dcc9a8 70%, #e8b87a 100%);
		animation: skyBreathe 14s ease-in-out infinite alternate;
	}

	.sunGlow {
		position: absolute;
		left: 50%;
		bottom: 18%;
		z-index: -2;
		width: min(70vw, 520px);
		aspect-ratio: 1;
		translate: -50% 40%;
		border-radius: 50%;
		background: radial-gradient(
			circle,
			color-mix(in oklab, var(--glow) 80%, white) 0%,
			transparent 68%
		);
		filter: blur(8px);
		opacity: 0.85;
		animation: glowPulse 8s ease-in-out infinite alternate;
	}

	.aspens {
		pointer-events: none;
		position: absolute;
		inset: 0;
		z-index: 2;
		overflow: hidden;
	}

	.leaf {
		position: absolute;
		top: -10%;
		left: var(--left);
		width: var(--size);
		height: calc(var(--size) * 1.35);
		background: var(--tone);
		opacity: 0.82;
		border-radius: 2% 80% 10% 80%;
		box-shadow: inset -2px -2px 0 color-mix(in oklab, black 12%, transparent);
		transform-origin: 50% 20%;
		animation: tumble var(--duration) linear var(--delay) infinite;
	}

	.leaf::after {
		content: '';
		position: absolute;
		inset: 18% 46% 8% 48%;
		background: color-mix(in oklab, black 18%, transparent);
		border-radius: 999px;
	}

	.ridges {
		position: absolute;
		left: -5%;
		bottom: 0;
		z-index: -1;
		width: 110%;
		height: clamp(200px, 38vh, 440px);
		pointer-events: none;
		overflow: hidden;
	}

	.ridge {
		animation: ridgeDrift 22s ease-in-out infinite alternate;
	}

	.ridge.far {
		fill: #7a9aab;
		opacity: 0.65;
		animation-duration: 30s;
	}

	.ridge.mid {
		fill: color-mix(in oklab, #3d5f4f 82%, var(--aspen));
		animation-duration: 22s;
		animation-delay: -5s;
	}

	.ridge.near {
		fill: #183028;
		animation-duration: 16s;
		animation-delay: -2s;
	}

	.content {
		position: relative;
		z-index: 1;
		width: min(100%, 920px);
		text-align: center;
		margin-block-end: clamp(3rem, 12vh, 8rem);
	}

	.eyebrow {
		margin: 0;
		font-size: clamp(0.85rem, 1.6vw, 1rem);
		letter-spacing: 0.22em;
		text-transform: uppercase;
		color: var(--ink-soft);
		animation: riseIn 900ms ease both;
	}

	.brand {
		margin: 0.15em 0 0;
		font-family: 'Fraunces', serif;
		font-optical-sizing: auto;
		font-weight: 700;
		font-size: clamp(3.2rem, 12vw, 7.5rem);
		line-height: 0.92;
		letter-spacing: -0.03em;
		color: var(--ink);
		text-wrap: balance;
		animation: riseIn 1100ms ease both;
	}

	.lede {
		margin: 0.9rem auto 0;
		max-width: 28ch;
		font-size: clamp(1rem, 2.2vw, 1.25rem);
		line-height: 1.45;
		color: var(--ink-soft);
		animation: riseIn 1300ms ease both;
	}

	.clock {
		display: grid;
		grid-template-columns: 1fr auto 1fr auto 1fr auto 1fr;
		align-items: end;
		gap: clamp(0.35rem, 1.5vw, 0.85rem);
		margin-block-start: clamp(1.75rem, 5vh, 2.75rem);
		animation: riseIn 1500ms ease both;
	}

	.unit {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.35rem;
		min-width: 0;
	}

	.value {
		font-variant-numeric: tabular-nums;
		font-weight: 600;
		font-size: clamp(2.4rem, 9vw, 5.5rem);
		line-height: 1;
		letter-spacing: -0.04em;
		color: var(--ink);
	}

	.label {
		font-size: clamp(0.65rem, 1.4vw, 0.8rem);
		letter-spacing: 0.16em;
		text-transform: uppercase;
		color: var(--ink-soft);
	}

	.sep {
		font-size: clamp(1.8rem, 6vw, 3.5rem);
		line-height: 1;
		color: color-mix(in oklab, var(--ink) 35%, transparent);
		transform: translateY(-0.35em);
	}

	.arrivedLine {
		margin: clamp(1.75rem, 5vh, 2.75rem) 0 0;
		font-family: 'Fraunces', serif;
		font-size: clamp(2rem, 6vw, 3.5rem);
		font-weight: 700;
		letter-spacing: -0.02em;
		animation: riseIn 700ms ease both;
	}

	.pulse {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 0.35rem 0.55rem;
		margin: clamp(1.4rem, 3.5vh, 2rem) 0 0;
		font-size: clamp(0.8rem, 1.7vw, 0.95rem);
		color: var(--ink-soft);
		animation: riseIn 1700ms ease both;
	}

	.dot {
		opacity: 0.55;
	}

	.arrived .sunGlow {
		opacity: 1;
		animation-duration: 4s;
	}

	.arrived .sky {
		filter: saturate(1.08) brightness(1.04);
	}

	@keyframes skyBreathe {
		from {
			filter: saturate(0.95) brightness(0.98);
		}
		to {
			filter: saturate(1.08) brightness(1.04);
		}
	}

	@keyframes glowPulse {
		from {
			opacity: 0.7;
			scale: 0.96;
		}
		to {
			opacity: 0.95;
			scale: 1.05;
		}
	}

	@keyframes tumble {
		from {
			transform: translate3d(0, -10vh, 0) rotate(0deg);
			opacity: 0;
		}
		8% {
			opacity: 0.9;
		}
		50% {
			transform: translate3d(calc(var(--drift) * 0.55), 50vh, 0) rotate(calc(var(--spin) * 0.5))
				translateX(var(--sway));
			opacity: 0.85;
		}
		to {
			transform: translate3d(var(--drift), 110vh, 0) rotate(var(--spin));
			opacity: 0.2;
		}
	}

	@keyframes ridgeDrift {
		from {
			transform: translateX(0);
		}
		to {
			transform: translateX(-1%);
		}
	}

	@keyframes riseIn {
		from {
			opacity: 0;
			transform: translateY(14px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@media (min-width: 641px) {
		:global(html),
		:global(body) {
			overflow: hidden;
		}

		.scene {
			height: 100dvh;
			min-height: 100dvh;
			max-height: 100dvh;
		}

		.content {
			margin-block-end: clamp(3rem, 11vh, 7rem);
		}
	}

	@media (max-width: 640px) {
		.clock {
			grid-template-columns: 1fr 1fr;
			gap: 1.25rem 1.5rem;
		}

		.sep {
			display: none;
		}

		.content {
			margin-block-end: clamp(5rem, 18vh, 8rem);
		}
	}

	@media (prefers-reduced-motion: reduce) {
		.sky,
		.sunGlow,
		.leaf,
		.ridge,
		.eyebrow,
		.brand,
		.lede,
		.clock,
		.arrivedLine,
		.pulse {
			animation: none;
		}
	}
</style>
