<script lang="ts">
	import { type LinksPage, type PlannerSettings, intersect } from '$lib';
	import Page from '$lib/components/Page.svelte';
	import SideNav from './SideNav.svelte';
	import TopNav from './TopNav.svelte';

	let { linkPage = {} as LinksPage, settings = {} as PlannerSettings } = $props();
	const year = $derived(settings.years[0]);
</script>

{#if linkPage}
	{#if linkPage.numPages}
		{#each new Array(linkPage.numPages) as _, itemPage (itemPage)}
			{@const id1 = linkPage.id}
			{@const id3 = itemPage === 0 ? '' : `pg${itemPage + 1}`}
			{@const id = [id1, id3].filter(Boolean).join('-')}
			<article {id} use:intersect={{ rootMargin: '1000px 0px 1000px 0px' }}>
				<SideNav
					tabs={!settings.monthPage.disable ? 'months' : 'none'}
					{settings}
					timeframe={year}
					disableActiveIndicator />
				<TopNav
					{settings}
					breadcrumbs={[
						{ name: `${linkPage.icon} ${linkPage.name}`, href: `#${linkPage.id}` },
					]} />
				<Page display={linkPage.type} {settings} timeframe={year} />
			</article>
		{/each}
	{/if}
{/if}

<style lang="scss">
	article {
		display: flex;
		align-items: center;
		flex-direction: column;
		padding-left: var(--sidenav-width);
		padding-top: var(--topnav-height);
	}
	:global(main.side-nav-right) article {
		padding-right: var(--sidenav-width);
		padding-left: 0;
	}
</style>
