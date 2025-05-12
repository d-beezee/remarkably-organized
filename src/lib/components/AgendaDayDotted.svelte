<script>
	import Grid from './Grid.svelte';
</script>

<div class="day">
	<div class="generic">
		<Grid display="dotted-small" />
	</div>
	<div class="hourly">
		{#each new Array(12) as _, i (i)}
			<div class="hour-label col-1">
				{i.toString().padStart(2, '0')}:00
			</div>
		{/each}
		{#each new Array(12) as _, i (i)}
			<div class="hour"></div>
		{/each}
		{#each new Array(12) as _, i (i)}
			<div class="hour-label col-2">
				{(i + 12).toString().padStart(2, '0')}:00
			</div>
		{/each}
		{#each new Array(12) as _, i (i)}
			<div class="hour"></div>
		{/each}
	</div>
</div>

<style lang="scss">
	.day {
		width: 100%;
		height: 100%;
		justify-items: stretch;
		align-items: stretch;
		grid-auto-flow: column;
		padding: 1rem 1rem 0 0;
		.generic {
			opacity: 0.3;
			width: 100%;
			height: 60%;
			margin-bottom: 1rem;
		}
		.hourly {
			display: grid;
			grid-template-columns: repeat(12, 1fr);
			grid-template-columns: 2.5rem 1fr 2.5rem 1fr;
			width: 100%;
			height: 40%;
			justify-items: stretch;
			align-items: stretch;
			grid-auto-flow: column;
		}
	}
	.hour {
		border-top: solid 1px var(--outline);
	}
	.hour-label {
		text-align: center;
		&.col-1 {
			grid-column: 1;
		}
		&.col-2 {
			grid-column: 3;
		}
		font-weight: var(--font-weight-light);
		font-size: 0.7em;
		color: var(--text-low);
		margin-top: -0.5rem;
		small {
			color: currentColor;
			font-size: 0.6em;
		}
	}
</style>
