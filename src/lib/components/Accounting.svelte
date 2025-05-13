<div class="container">
	<div class="task-progress">
		<div class="header">
			<div>Spesa</div>
			<div>€</div>
		</div>
		{#each new Array(40) as _, i (i)}
			<div class="row">
				<div class="task-input"></div>
				<div class="priority">
					<div>_______ , ___ €</div>
				</div>
			</div>
		{/each}
	</div>
	<div class="task-progress">
		<div class="header">
			<div>Spesa</div>
			<div>€</div>
		</div>
		{#each new Array(40) as _, i (i)}
			<div class="row">
				<div class="task-input"></div>
				<div class="priority">
					<div>_______ , ___ €</div>
				</div>
			</div>
		{/each}
	</div>
</div>

<style lang="scss">
	.container {
		display: flex;
	}
	.task-progress {
		display: grid;
		grid-template-rows: auto repeat(25, 1fr);
		gap: 0; /* Remove gap between rows */
		padding: 1rem;
		width: 50%;
		height: 100%;

		.header {
			display: grid;
			grid-template-columns: 4fr 2fr; /* Adjusted to give more space for tasks */
			font-weight: bold;
			text-align: center;
			padding-bottom: 0.5rem;
			border-bottom: 1px solid var(--outline);
		}

		.row {
			display: grid;
			grid-template-columns: 4fr 2fr; /* Adjusted to give more space for tasks */
			align-items: center;
			gap: 0; /* Remove gap between elements in row */
			height: 1.2rem; /* Condense row height */
			border-bottom: 1px solid var(--outline);

			.task-input {
				width: 100%;
				border: none;
				font-size: 1em;
				outline: none;
				padding: 0.25rem 0;
			}

			.priority {
				display: flex;
				justify-content: center;
				bottom: -5px;
				position: relative;
			}
		}
	}
</style>
