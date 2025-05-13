<script lang="ts">
	import type { PlannerSettings } from '$lib';
	import Grid from './Grid.svelte';

	let { settings = {} as PlannerSettings } = $props();
	const font = $derived(settings.design.font);
	const height = $derived(settings.linksPage.size);
</script>

<div class="links-page">
	<div class="links">
		{#each settings.linksPages as link, i (link.name)}
			<a href="#{link.id}">
				<div class="title-block">
					<div class="decor-line top"><span>{link.icon}​</span></div>
					<h2 style:font-family="'{font}'">{link.name}</h2>
					<div class="decor-line"></div>
				</div>
			</a>
		{/each}
	</div>
	<div class="grid">
		<Grid display="dotted" />
	</div>
</div>

<style lang="scss">
	.links-page {
		height: 100%;
		.links {
			display: flex;
			align-items: center;
			justify-content: space-around;
			height: var(--links-page-height);
			.title-block {
				text-align: center;
				margin-bottom: 40px;
			}

			.decor-line {
				display: inline-block;
				font-size: 20px;
				color: #aaa;
				margin-bottom: 10px;
				position: relative;
				&::before,
				&::after {
					content: '';
					display: inline-block;
					width: 50px;
					height: 1px;
					background: #ccc;
					vertical-align: middle;
				}
				&.top::before,
				&.top::after {
					margin: 0 12px;
				}
			}
		}
		.grid {
			height: var(--links-page-remaining-height);
		}
	}
</style>
