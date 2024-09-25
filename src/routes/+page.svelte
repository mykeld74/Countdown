<script>
	// import '$lib/css/reset.css';
	import 'open-props/style';
	import 'open-props/normalize';
	let timeLeft;
	let daysLeft;
	let hoursLeft;
	let minutesLeft;
	let secondsLeft;

	const endTime = new Date('2024-09-25T15:00:00').getTime();

	const countdown = () => {
		timeLeft = Math.floor((endTime - Date.now()) / 1000);
		daysLeft = Math.floor(timeLeft / (24 * 60 * 60));
		hoursLeft = Math.floor((timeLeft / (60 * 60)) % 24);
		minutesLeft = Math.floor((timeLeft / 60) % 60);
		secondsLeft = timeLeft % 60;

		if (timeLeft <= 0) {
			clearInterval(countdownInterval);
			timeLeft = 0;
		}
	};

	const countdownInterval = setInterval(countdown, 100);
</script>

<div class="countdown-container">
	<div class="title-container"><h1 class="countdown-title">Countdown to Breckenridge</h1></div>
	{#if timeLeft > 0}
		<div class="countdown">
			<div class={daysLeft === 0 ? 'countdown-item zero' : 'countdown-item'}>
				<p class="countdown-number">{daysLeft}</p>
				<p class="countdown-text">Days</p>
			</div>
			<div class={daysLeft === 0 && hoursLeft === 0 ? 'countdown-item  zero' : 'countdown-item'}>
				<p class="countdown-number">{hoursLeft}</p>
				<p class="countdown-text">Hours</p>
			</div>
			<div
				class={daysLeft === 0 && hoursLeft === 0 && minutesLeft === 0
					? 'countdown-item zero'
					: 'countdown-item '}
			>
				<p class="countdown-number">{minutesLeft}</p>
				<p class="countdown-text">Minutes</p>
			</div>
			<div class="countdown-item">
				<p class="countdown-number">{secondsLeft}</p>
				<p class="countdown-text">Seconds</p>
			</div>
		</div>
	{/if}
	{#if timeLeft <= 0}
		<div class="outtaHere">
			<p class="countdown-number">We're Outta Here!</p>
		</div>
	{/if}
</div>

<style>
	:global(body) {
		background-color: var(--surface-3);
	}
	.countdown-container {
		container-type: inline-size;
		display: grid;
		place-items: center;
		min-height: 100vh;
		grid-template-rows: fit-content(50px) 1fr;
	}
	.countdown {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 1rem;
		margin-block-start: -10em;
		@container (max-width: 600px) {
			grid-template-columns: 1fr 1fr;
			gap: 3rem;
		}
	}
	.countdown-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background-color: var(--surface-2);
		padding: 1rem;
		border-radius: 0.5rem;
		box-shadow: var(--shadow-5);
		&.zero {
			color: oklch(38.5% 0.0874 21.23);
			opacity: 0.5;
			scale: 0.95;
		}
	}
	.countdown-number {
		font-size: clamp(3rem, 10vw, 10rem);
		font-weight: bold;
		margin-block: 0;
	}
	.countdown-text {
		font-size: clamp(1.5rem, 4vw, 4rem);
		font-weight: bold;
		margin-block: 0 0.5em;
	}
	.countdown-title {
		font-size: clamp(2rem, 4vw, 4rem);
		font-weight: bold;
		margin-block: 0;
		padding-block: 20px 20px;
		text-align: center;
	}
	.title-container {
		display: flex;
		width: 100%;
		border-block-end: 1px solid var(--surface-4);
		background: var(--surface-1);
		align-items: center;
		justify-content: center;
	}
	.outtaHere {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		margin-block-start: -12em;
	}
</style>
