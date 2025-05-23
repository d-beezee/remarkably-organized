<script lang="ts">
	import { browser } from '$app/environment';
	import { replaceState } from '$app/navigation';
	import { page } from '$app/state';
	import Toast from '$lib/components/Toast.svelte';
	import { slide } from 'svelte/transition';
	import LoadingIcon from '~icons/eos-icons/bubble-loading';
	import SettingsIcon from '~icons/fa/cog';
	import { fonts, getGoogleFontURL } from '../fonts/fonts';
	import CollectionPages from './CollectionPages.svelte';
	import CoverPage from './CoverPage.svelte';
	import DayPage from './DayPage.svelte';
	import HelpModal from './HelpModal.svelte';
	import LinksPage from './LinksPage.svelte';
	import LinksPages from './LinksPages.svelte';
	import MonthPage from './MonthPage.svelte';
	import QuarterPage from './QuarterPage.svelte';
	import WeekPage from './WeekPage.svelte';
	import YearPage from './YearPage.svelte';
	let { data } = $props();
	const { settings } = data;

	const pageTemplates = [
		{ name: 'Blank Page', value: 'blank' },
		{ name: 'Dotted Grid - Small', value: 'dotted-small' },
		{ name: 'Dotted Grid - Medium', value: 'dotted' },
		{ name: 'Dotted Grid - Large', value: 'dotted-large' },
		{ name: 'Grid - Small', value: 'grid-small' },
		{ name: 'Grid - Medium', value: 'grid' },
		{ name: 'Grid - Large', value: 'grid-large' },
		{ name: 'Lined - Small', value: 'lined-small' },
		{ name: 'Lined - Medium', value: 'lined' },
		{ name: 'Lined - Large', value: 'lined-large' },
		{ name: 'Numbered - Small', value: 'numbered-small' },
		{ name: 'Numbered - Medium', value: 'numbered' },
		{ name: 'Numbered - Large', value: 'numbered-large' },
		{ name: 'To-do List - Small', value: 'todo-small' },
		{ name: 'To-do List - Medium', value: 'todo' },
		{ name: 'To-do List - Large', value: 'todo-large' },
		{ name: 'Task List - Progress', value: 'tasklist-progress' },
		{ name: 'Calendar', value: 'calendar-month' },
		{ name: 'Calendar - With Notes', value: 'calendar-month-with-notes' },
		{ name: 'Agenda - Daily', value: 'agenda-day' },
		{ name: 'Agenda - Daily Dotted	', value: 'agenda-day-dotted' },
		{ name: 'Agenda - Weekly', value: 'agenda-week' },
		{ name: 'Notes - Yearly', value: 'notes-year' },
		{ name: 'Notes - Quarterly', value: 'notes-quarter' },
		{ name: 'Notes - Weekly', value: 'notes-week' },
		{ name: 'Notes - Weekly - Columns', value: 'notes-week-columns' },
		{ name: 'Notes - Weekly - Rows', value: 'notes-week-rows' },
		{ name: 'Notes - Daily', value: 'notes-day' },
		{ name: 'Habit Checkboxes - Grouped by Week', value: 'habit-year-by-week' },
		{ name: 'Habit Checkboxes - Grouped by Month', value: 'habit-year-by-month' },
		{ name: 'School Timetable', value: 'school-timetable' },
		{ name: 'Accounting', value: 'accounting' },
	];

	const font = $derived(fonts.find((f) => f.name === settings.design.font) ?? fonts[0]);
	const googleFontURL = $derived(
		getGoogleFontURL([
			settings.design.font,
			settings.coverPage.font,
			settings.topNav.font,
			settings.sideNav.font,
		]),
	);
	const googleFontImport = $derived(
		googleFontURL ? `@import url("${googleFontURL}");` : '',
	);

	function getAvailablePageTemplates(
		location: 'collection' | 'year' | 'month' | 'quarter' | 'week' | 'day',
	) {
		return pageTemplates.filter((t) => {
			if (
				!t.value.startsWith('notes') &&
				!t.value.startsWith('habit') &&
				!t.value.startsWith('calendar')
			) {
				return true;
			}
			if (location === 'collection') {
				return ![
					'notes-quarter',
					'notes-month',
					'calendar-month',
					'calendar-month-with-notes',
				].includes(t.value);
			}
			const timeframe = t.value.split('-')[1];
			return location === timeframe;
		});
	}

	let customTimeframe = $state(false);
	let showHelp = $state(page.url.searchParams.get('help') !== '0');
	let showMenu = $state(true);
	let showAdvancedSettings = $state(false);
	let enableHighResolution = $state(page.url.searchParams.has('highres'));
	let loadPages = $state(
		page.url.searchParams.get('help') === '0' &&
			(browser || page.url.searchParams.get('load') === '1'),
	);

	let settingsUrlInitialized = false;
	$effect(() => {
		const url = new URL(document.location.href);
		if (settings.edits) {
			url.searchParams.set('settings', JSON.stringify(settings.edits));
			replaceState(url, {});
		} else if (settingsUrlInitialized) {
			url.searchParams.delete('settings');
			replaceState(url, {});
		}
		settingsUrlInitialized = true;
	});
	$effect(() => {
		const url = new URL(document.location.href);
		if (enableHighResolution && !url.searchParams.has('highres')) {
			url.searchParams.set('highres', '');
			replaceState(url, {});
		}
		if (!enableHighResolution && url.searchParams.has('highres')) {
			url.searchParams.delete('highres');
			replaceState(url, {});
		}
	});

	const debounces = new Map<string, number>();
	function debounce(key: string, wait: number, func: () => void) {
		const later = () => {
			debounces.set(key, 0);
			func();
		};
		clearTimeout(debounces.get(key));
		debounces.set(key, setTimeout(later, wait));
	}

	function onStartDateChange(e: Event) {
		const target = e.target as HTMLInputElement;
		if (!target.value) return;
		try {
			const date = new Date(target.value);
			debounce('onStartDateChange', 1000, () => {
				date.setUTCHours(0, 0, 0, 0);
				if (date.getTime()) settings.date.start = date;
			});
		} catch (error) {
			// ignore
		}
	}
	function onEndDateChange(e: Event) {
		const target = e.target as HTMLInputElement;
		if (!target.value) return;
		try {
			const date = new Date(target.value);
			debounce('onEndDateChange', 1000, () => {
				date.setUTCHours(0, 0, 0, 0);
				if (date.getTime()) settings.date.end = date;
			});
		} catch (error) {
			// ignore
		}
	}

	function onHelpClose() {
		showHelp = false;
		const url = new URL(document.location.href);
		url.searchParams.set('help', '0');
		replaceState(url, {});
		setTimeout(() => (loadPages = true), 180);
	}

	function onTimeframeSelection(e: Event) {
		const target = e.target as HTMLSelectElement;
		if (!target.value) return;
		const startTime = +target.value;
		if (!startTime) {
			customTimeframe = true;
			return;
		}
		const newStart = new Date(startTime);
		const newEnd = new Date(Date.UTC(newStart.getUTCFullYear(), 11, 31));
		if (
			newStart.getTime() !== settings.date.start.getTime() ||
			newEnd.getTime() !== settings.date.end.getTime()
		) {
			settings.date.start = newStart;
			settings.date.end = newEnd;
		}
		customTimeframe = false;
	}

	// Update the page printing resolution
	$effect(() => {
		let element = document.getElementById('page-resolution-style');
		if (!element) {
			element = document.createElement('style');
			element.id = 'page-resolution-style';
			document.head.appendChild(element);
		}
		element.innerHTML = `@page {${
			enableHighResolution ? 'size: 1404px 1872px;' : 'size: 702px 936px;'
		}margin: 0;}`;
	});
</script>

<svelte:head>
	<title>Planner Builder | Remarkably Organized</title>
	{#if googleFontImport}
		{@html `<style type="text/css">${googleFontImport}</style>`}
	{/if}
</svelte:head>

{#if showHelp}<HelpModal onClose={onHelpClose} />{/if}

{#if showMenu}
	<div class="menu" transition:slide={{ duration: 200 }}>
		<h2>Settings</h2>
		<form>
			<fieldset>
				<label for="timeframeBasedOnYear">Year</label>
				<select
					id="timeframeBasedOnYear"
					value={settings.date.start.getUTCMonth() === 0 &&
					settings.date.start.getUTCDate() === 1 &&
					settings.date.end.getUTCMonth() === 11 &&
					settings.date.end.getUTCDate() === 31 &&
					!customTimeframe
						? settings.date.start.getTime()
						: 0}
					onchange={onTimeframeSelection}>
					{#each new Array(7) as _, i (i)}
						{@const date = new Date(Date.UTC(new Date().getFullYear() - 1 + i))}
						<option value={date.getTime()}>
							{date.getUTCFullYear()}
						</option>
					{/each}
					<option value={0}>Custom Date Range</option>
				</select>
			</fieldset>
			{#if customTimeframe || settings.date.start.getUTCMonth() !== 0 || settings.date.start.getUTCDate() !== 1 || settings.date.end.getUTCMonth() !== 11 || settings.date.end.getUTCDate() !== 31}
				<fieldset>
					<label for="start">Start Date</label>
					<input
						type="date"
						placeholder="Start Date"
						id="start"
						max={settings.date.end.toISOString().slice(0, 10)}
						value={settings.date.start.toISOString().slice(0, 10)}
						onchange={onStartDateChange} />
				</fieldset>
				<fieldset>
					<label for="end">End Date</label>
					<input
						type="date"
						placeholder="End Date"
						id="end"
						min={settings.date.start.toISOString().slice(0, 10)}
						value={settings.date.end.toISOString().slice(0, 10)}
						onchange={onEndDateChange} />
				</fieldset>
			{/if}
			<div class="checkbox">
				<input
					type="checkbox"
					bind:checked={settings.date.startWeekOnSunday}
					id="startWeekOnSunday" />
				<label for="startWeekOnSunday">Start Week on Sunday</label>
			</div>
			<div class="checkbox">
				<input
					type="checkbox"
					bind:checked={enableHighResolution}
					id="enableHighResolution" />
				<label for="enableHighResolution">Print in high resolution (bigger file)</label>
			</div>

			{#if showAdvancedSettings}
				<h3>Cover Page</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.coverPage.disable}
						id="disableCoverPage" />
					<label for="disableCoverPage">Disable Cover Page</label>
				</div>
				{#if !settings.coverPage.disable}
					<fieldset>
						<label for="coverPageTitle">Cover Page Title</label>
						<input
							type="text"
							placeholder="Cover Page Title"
							id="coverPageTitle"
							bind:value={settings.coverPage.title} />
					</fieldset>
					<fieldset>
						<label for="name">Contact Name</label>
						<input
							type="text"
							placeholder="Name"
							id="name"
							bind:value={settings.coverPage.name} />
					</fieldset>
					<fieldset>
						<label for="email">Contact Email/Phone</label>
						<input
							type="text"
							placeholder="Contact Email/Phone"
							id="email"
							bind:value={settings.coverPage.email} />
					</fieldset>
					<fieldset>
						<label for="coverPageFont">Font</label>
						<select id="coverPageFont" bind:value={settings.coverPage.font}>
							{#each fonts as font (font.name)}
								<option value={font.name}>{font.name}</option>
							{/each}
						</select>
					</fieldset>
					<div class="checkbox">
						<input
							type="checkbox"
							bind:checked={settings.coverPage.showCollectionLinks}
							id="coverPageShowCollectionLinks" />
						<label for="coverPageShowCollectionLinks">Show Links to Collections</label>
					</div>
					<div class="checkbox">
						<input
							type="checkbox"
							bind:checked={settings.coverPage.darkBackground}
							id="coverPageDarkBackground" />
						<label for="coverPageDarkBackground">Dark Background</label>
					</div>
				{/if}

				<h3>Links Page</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.linksPage.disable}
						id="disableLinksPage" />
					<label for="disableLinksPage">Disable Links Page</label>

					<input
						type="checkbox"
						bind:checked={settings.linksPage.showFullLinks}
						id="showFullLinks" />
					<label for="showFullLinks">Show full links</label>

					<fieldset style="margin-top: 1rem;">
						<label for="linkspage-height">Index page button part height</label>
						<input
							type="number"
							placeholder="Number of Pages"
							id="linkspage-height"
							min="0.1"
							max="1"
							step="0.1"
							bind:value={settings.linksPage.size} />
					</fieldset>
				</div>
				{#if !settings.linksPage.disable}
					{#each settings.linksPages as linksPage, i (linksPage.id)}
						<fieldset>
							<label for="">Link {i + 1}</label>
							<input type="text" bind:value={linksPage.name} placeholder="Name" />

							<fieldset style="margin-top: 1rem;">
								<label for="linkspage-{linksPage.id}-numPages">Number of Pages</label>
								<input
									type="number"
									placeholder="Number of Pages"
									id="linkspage-{linksPage.id}-numPages"
									min="1"
									step="1"
									bind:value={linksPage.numPages} />
							</fieldset>
							<fieldset style="margin-top: 1rem;">
								<label for="linkspage-{linksPage.id}-numPages">Icon</label>
								<input
									type="text"
									placeholder="Icon"
									id="linkspage-{linksPage.id}-icon"
									bind:value={linksPage.icon} />
							</fieldset>
							<fieldset style="margin-top: 1rem;">
								<label for="collection-{linksPage.id}-type">Page Template</label>
								<select id="collection-{linksPage.id}-type" bind:value={linksPage.type}>
									{#each getAvailablePageTemplates('collection') as template}
										<option value={template.value}>{template.name}</option>
									{/each}
								</select>
							</fieldset>

							<button
								type="button"
								onclick={() => settings.linksPages.splice(i, 1)}
								style:color="var(--error)">
								Remove Link
							</button>
						</fieldset>
					{/each}
					<button
						type="button"
						onclick={() =>
							settings.linksPages.push({
								name: 'Page',
								id: `${Date.now()}`,
								numPages: 1,
								icon: '§				',
								type: 'dotted',
							})}>
						Add New Link
					</button>
				{/if}

				<h3>Yearly View</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.yearPage.disable}
						id="yearPageDisable" />
					<label for="yearPageDisable">Disable Yearly View</label>
				</div>
				{#if !settings.yearPage.disable}
					<fieldset>
						<label for="yearNotePagesAmount">Additional Note Pages</label>
						<input
							type="number"
							placeholder="Additional Note Pages"
							id="yearNotePagesAmount"
							min="0"
							step="1"
							bind:value={settings.yearPage.notePagesAmount} />
					</fieldset>
					{#if settings.yearPage.notePagesAmount > 0}
						<fieldset>
							<label for="yearNotePagesTemplate">Additional Note Pages Template</label>
							<select
								id="yearNotePagesTemplate"
								bind:value={settings.yearPage.notePagesTemplate}>
								{#each getAvailablePageTemplates('year') as template (template.value)}
									<option value={template.value}>{template.name}</option>
								{/each}
							</select>
						</fieldset>
					{/if}
				{/if}

				<h3>Quarterly View</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.quarterPage.disable}
						id="quarterPageDisable" />
					<label for="quarterPageDisable">Disable Quarterly View</label>
				</div>
				{#if !settings.quarterPage.disable}
					<fieldset>
						<label for="quarterNotePagesAmount">Additional Note Pages</label>
						<input
							type="number"
							placeholder="Additional Note Pages"
							id="quarterNotePagesAmount"
							min="0"
							step="1"
							bind:value={settings.quarterPage.notePagesAmount} />
					</fieldset>
					{#if settings.quarterPage.notePagesAmount > 0}
						<fieldset>
							<label for="quarterNotePagesTemplate">Additional Note Pages Template</label>
							<select
								id="quarterNotePagesTemplate"
								bind:value={settings.quarterPage.notePagesTemplate}>
								{#each getAvailablePageTemplates('quarter') as template (template.value)}
									<option value={template.value}>{template.name}</option>
								{/each}
							</select>
						</fieldset>
					{/if}
				{/if}

				<h3>Monthly View</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.monthPage.disable}
						id="monthPageDisable" />
					<label for="monthPageDisable">Disable Monthly View</label>
				</div>
				{#if !settings.monthPage.disable}
					<fieldset>
						<label for="monthPageTemplate">Month Page Template</label>
						<select id="monthPageTemplate" bind:value={settings.monthPage.template}>
							{#each getAvailablePageTemplates('month') as template (template.value)}
								<option value={template.value}>{template.name}</option>
							{/each}
						</select>
					</fieldset>
					<fieldset>
						<label for="monthNotePagesAmount">Additional Note Pages</label>
						<input
							type="number"
							placeholder="Additional Note Pages"
							id="monthNotePagesAmount"
							min="0"
							step="1"
							bind:value={settings.monthPage.notePagesAmount} />
					</fieldset>
					{#if settings.monthPage.notePagesAmount > 0}
						<fieldset>
							<label for="monthNotePagesTemplate">Additional Note Pages Template</label>
							<select
								id="monthNotePagesTemplate"
								bind:value={settings.monthPage.notePagesTemplate}>
								{#each getAvailablePageTemplates('month') as template (template.value)}
									<option value={template.value}>{template.name}</option>
								{/each}
							</select>
						</fieldset>
					{/if}
				{/if}

				<h3>Weekly View</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.weekPage.disable}
						id="weekPageDisable" />
					<label for="weekPageDisable">Disable Weekly View</label>
				</div>
				{#if !settings.weekPage.disable}
					<fieldset>
						<label for="weekPageTemplate">Week Page Template</label>
						<select id="weekPageTemplate" bind:value={settings.weekPage.template}>
							{#each getAvailablePageTemplates('week') as template (template.value)}
								<option value={template.value}>{template.name}</option>
							{/each}
						</select>
					</fieldset>
					<fieldset>
						<label for="weekNotePagesAmount">Additional Note Pages</label>
						<input
							type="number"
							placeholder="Additional Note Pages"
							id="weekNotePagesAmount"
							min="0"
							step="1"
							bind:value={settings.weekPage.notePagesAmount} />
					</fieldset>
					{#if settings.weekPage.notePagesAmount > 0}
						<fieldset>
							<label for="weekNotePagesTemplate">Additional Note Pages Template</label>
							<select
								id="weekNotePagesTemplate"
								bind:value={settings.weekPage.notePagesTemplate}>
								{#each getAvailablePageTemplates('week') as template (template.value)}
									<option value={template.value}>{template.name}</option>
								{/each}
							</select>
						</fieldset>
					{/if}
					<fieldset>
						<label for="sideNavDisplay">Sidebar Display</label>
						<select id="sideNavDisplay" bind:value={settings.weekPage.sideNavDisplay}>
							<option value="days-this-week">Days of the Week</option>
							<option value="days-this-month">Days of the Month</option>
							<option value="weeks-this-year">Weeks of the Year</option>
							<option value="weeks-this-month">Weeks of the Month</option>
							<option value="months">Months</option>
							<option value="none">None</option>
						</select>
					</fieldset>
					{#if settings.weekPage.sideNavDisplay === 'weeks-this-month' || settings.weekPage.sideNavDisplay === 'weeks-this-year'}
						<div class="checkbox">
							<input
								type="checkbox"
								bind:checked={settings.weekPage.useWeekNumbersInSideNav}
								id="useWeekNumbersInSideNav" />
							<label for="useWeekNumbersInSideNav">Show week numbers in side bar</label>
						</div>
					{/if}
					<div class="checkbox">
						<input
							type="checkbox"
							bind:checked={settings.weekPage.useWeekSinceYear}
							id="useWeekSinceYear" />
						<label for="useWeekSinceYear">Use week number from start of year</label>
					</div>
				{/if}

				<h3>Daily View</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.dayPage.disable}
						id="dayPageDisable" />
					<label for="dayPageDisable">Disable Daily View</label>
				</div>
				{#if !settings.dayPage.disable}
					<fieldset>
						<label for="dayPageTemplate">Day Page Template</label>
						<select id="dayPageTemplate" bind:value={settings.dayPage.template}>
							{#each getAvailablePageTemplates('day') as template (template.value)}
								<option value={template.value}>{template.name}</option>
							{/each}
						</select>
					</fieldset>
					<fieldset>
						<label for="dayNotePagesAmount">Additional Note Pages</label>
						<input
							type="number"
							placeholder="Additional Note Pages"
							id="dayNotePagesAmount"
							min="0"
							step="1"
							bind:value={settings.dayPage.notePagesAmount} />
					</fieldset>
					{#if settings.dayPage.notePagesAmount > 0}
						<fieldset>
							<label for="dayNotePagesTemplate">Additional Note Pages Template</label>
							<select
								id="dayNotePagesTemplate"
								bind:value={settings.dayPage.notePagesTemplate}>
								{#each getAvailablePageTemplates('day') as template (template.value)}
									<option value={template.value}>{template.name}</option>
								{/each}
							</select>
						</fieldset>
					{/if}
					<fieldset>
						<label for="sideNavDisplay">Sidebar Display</label>
						<select id="sideNavDisplay" bind:value={settings.dayPage.sideNavDisplay}>
							<option value="days-this-week">Days of the Week</option>
							<option value="days-this-month">Days of the Month</option>
							<option value="days-this-year">Days of the Year</option>
							<option value="weeks-this-year">Weeks of the Year</option>
							<option value="weeks-this-month">Weeks of the Month</option>
							<option value="months">Months</option>
							<option value="none">None</option>
						</select>
					</fieldset>
				{/if}

				<h3>Design</h3>
				<fieldset>
					<label for="designFont">Font</label>
					<select id="designFont" bind:value={settings.design.font}>
						{#each fonts as font (font.name)}
							<option value={font.name}>{font.name}</option>
						{/each}
					</select>
				</fieldset>
				<fieldset>
					<label for="start">Text Color</label>
					<input
						type="color"
						placeholder="Text Color"
						id="start"
						bind:value={settings.design.colorText} />
				</fieldset>
				<fieldset>
					<label for="start">Lines/Border Color</label>
					<input
						type="color"
						placeholder="Lines/Border"
						id="start"
						bind:value={settings.design.colorLines} />
				</fieldset>
				<fieldset>
					<label for="start">Dots Color</label>
					<input
						type="color"
						placeholder="Lines/Border"
						id="start"
						bind:value={settings.design.colorDots} />
				</fieldset>

				<h3>Sidebar Navigation</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.sideNav.disable}
						id="sideNavDisable" />
					<label for="sideNavDisable">Disable Sidebar</label>
				</div>
				{#if !settings.sideNav.disable}
					<div class="checkbox">
						<input
							type="checkbox"
							bind:checked={settings.sideNav.leftSide}
							id="sideNavLeftSide" />
						<label for="sideNavLeftSide">Show Sidebar on Left</label>
					</div>
					<div class="checkbox">
						<input
							type="checkbox"
							bind:checked={settings.sideNav.showCollectionLinks}
							id="sideNavShowCollectionLinks" />
						<label for="sideNavShowCollectionLinks">Show Links to Collections</label>
					</div>
					<fieldset>
						<label for="sideNavFont">Font</label>
						<select id="sideNavFont" bind:value={settings.sideNav.font}>
							{#each fonts as font (font.name)}
								<option value={font.name}>{font.name}</option>
							{/each}
						</select>
					</fieldset>
				{/if}

				<h3>Topbar Navigation</h3>
				<div class="checkbox">
					<input
						type="checkbox"
						bind:checked={settings.topNav.disable}
						id="topNavDisable" />
					<label for="topNavDisable">Disable Topbar</label>
				</div>
				{#if !settings.topNav.disable}
					<div class="checkbox">
						<input
							type="checkbox"
							bind:checked={settings.topNav.showCollectionLinks}
							id="topNavShowCollectionLinks" />
						<label for="topNavShowCollectionLinks">Show Links to Collections</label>
					</div>
					<fieldset>
						<label for="topNavFont">Font</label>
						<select id="topNavFont" bind:value={settings.topNav.font}>
							{#each fonts as font (font.name)}
								<option value={font.name}>{font.name}</option>
							{/each}
						</select>
					</fieldset>
				{/if}

				<h3>Collections</h3>
				<div class="collections">
					{#each settings.collections as collection, i (collection.id)}
						<fieldset>
							<label for="">Collection {i + 1}</label>
							<input type="text" bind:value={collection.name} placeholder="Name" />
							<fieldset style="margin-top: 1rem;">
								<label for="collection-{collection.id}-type">Page Template</label>
								<select id="collection-{collection.id}-type" bind:value={collection.type}>
									{#each getAvailablePageTemplates('collection') as template}
										<option value={template.value}>{template.name}</option>
									{/each}
								</select>
							</fieldset>
							{#if collection.type.startsWith('numbered') || collection.type.startsWith('lined') || collection.type.startsWith('todo')}
								<fieldset style="margin-top: 1rem;">
									<label for="collection-{collection.id}-columns">Columns</label>
									<input
										type="number"
										placeholder="Columns"
										id="collection-{collection.id}-columns"
										min="1"
										step="1"
										bind:value={collection.columns} />
								</fieldset>
							{/if}
							<fieldset style="margin-top: 1rem;">
								<label for="collection-{collection.id}-numIndexPages">
									Number of Index Pages
								</label>
								<input
									type="number"
									placeholder="Number of Index Pages"
									id="collection-{collection.id}-numIndexPages"
									min="0"
									step="1"
									bind:value={collection.numIndexPages} />
							</fieldset>
							<fieldset style="margin-top: 1rem;">
								<label for="collection-{collection.id}-total">
									Number of Items Per Index Page
								</label>
								<input
									type="number"
									placeholder="Number of Items Per Index Page"
									id="collection-{collection.id}-total"
									min="1"
									max="180"
									step="1"
									bind:value={collection.total} />
							</fieldset>
							<fieldset style="margin-top: 1rem;">
								<label for="collection-{collection.id}-numPagesPerItem">
									Number of Pages Per Item
								</label>
								<input
									type="number"
									placeholder="Number of Pages Per Item"
									id="collection-{collection.id}-numPagesPerItem"
									min="1"
									step="1"
									bind:value={collection.numPagesPerItem} />
							</fieldset>
							<button
								type="button"
								onclick={() => settings.collections.splice(i, 1)}
								style:color="var(--error)">
								Remove Collection
							</button>
						</fieldset>
					{/each}
					<button
						type="button"
						onclick={() =>
							settings.collections.push({
								name: 'Notes',
								id: `${Date.now()}`,
								total: 20,
								type: 'blank',
								numIndexPages: 1,
								numPagesPerItem: 1,
								columns: 1,
							})}>
						Add New Collection
					</button>
				</div>

				<h3>Calendar Events</h3>
				{#each settings.calendars as calendar, i (calendar.url)}
					<h4 style="margin-top: 1rem;">
						{calendar.name || 'Custom Calendar'} ({calendar.events.length} Events)
						{#if calendar.lastUpdated}
							<p
								style="display: block; font-weight: normal; font-size: .8rem; opacity: .8">
								Last Updated: {new Date(calendar.lastUpdated).toLocaleString()}
							</p>
						{/if}
					</h4>
					<fieldset>
						<label for="calendar-{i}-url">Calendar URL (ICS file)</label>
						<input
							type="text"
							placeholder="Calendar URL (ICS file)"
							id="calendar-{i}-url"
							bind:value={calendar.url} />
					</fieldset>
					<div style="display: flex;">
						<button
							type="button"
							style="flex: 1"
							disabled={settings.calendars.some((calendar) => calendar.updating)}
							onclick={() => settings.importEvents(i)}>
							{calendar.updating ? `Importing...` : `Import Events`}
						</button>
						<button
							type="button"
							style="flex: 1"
							disabled={settings.calendars.some((calendar) => calendar.updating)}
							onclick={() => settings.calendars.splice(i, 1)}
							style:color="var(--error)">
							Remove Calendar
						</button>
					</div>
				{/each}
				<button
					type="button"
					disabled={settings.calendars.some((calendar) => calendar.updating)}
					onclick={() =>
						settings.calendars.push({
							events: [],
							lastUpdated: 0,
							name: '',
							updating: false,
							url: '',
						})}>
					Add New Calendar
				</button>
			{:else}
				<button
					type="button"
					style="margin: 1rem 0;"
					onclick={() => (showAdvancedSettings = true)}>
					Advanced Settings
				</button>
			{/if}
		</form>
		<div class="actions">
			<div>
				<button class="export" onclick={() => window.print()}>Print to PDF</button>
			</div>
			<div class="import-export">
				<button
					class="export-json"
					onclick={() => {
						const serialized = settings.serialize();
						// copy to clipboard
						navigator.clipboard.writeText(JSON.stringify(serialized));
					}}>
					Copy config
				</button>
				<button
					class="import-json"
					onclick={() => {
						const result = window.prompt('Insert the config');
						if (!result) return;
						settings.set(result);
					}}>
					Import config
				</button>
			</div>
		</div>
	</div>
{/if}
<button onclick={() => (showMenu = !showMenu)} class="menu-trigger">
	<SettingsIcon />
</button>
<Toast />

<main
	style:--doc-width="{702}px"
	style:--doc-height="{702 * (1 / (settings.design.aspectRatio || 1))}px"
	style:--sidenav-width="{settings.sideNav.disable ? 0 : settings.sideNav.width}px"
	style:--topnav-height="{settings.topNav.disable ? 0 : settings.topNav.height}px"
	style:--font="'{font.name}'"
	style:--font-size="{font.size}rem"
	style:--font-weight-bold={font.boldWeight}
	style:--font-weight-normal={font.normalWeight}
	style:--font-weight-light={font.lightWeight}
	style:--links-page-height="{settings.linksPage.size * 100}%"
	style:--links-page-remaining-height="{100 - settings.linksPage.size * 100}%"
	style:--text={settings.design.colorText}
	style:--outline={settings.design.colorLines}
	style:--dots-color={settings.design.colorDots}
	style:font-size="{font.size}rem"
	class:side-nav-right={!settings.sideNav.leftSide}
	class:high-res={enableHighResolution}>
	<div id="home"></div>
	{#if !loadPages}
		<article
			style="display: flex; flex-direction: column; align-items: center; justify-content: center;">
			<h1 style="margin-bottom: 2rem;">Loading...</h1>
			<LoadingIcon font-size="3rem" />
		</article>
	{/if}
	{#if !settings.coverPage.disable && loadPages}
		<CoverPage {settings} />
	{/if}
	{#if !settings.linksPage.disable && loadPages}
		<LinksPage {settings} />
	{/if}
	{#if !settings.yearPage.disable && loadPages}
		{#each settings.years as year, i}
			<YearPage {settings} {year} />
		{/each}
	{/if}
	{#if !settings.quarterPage.disable && loadPages}
		{#each settings.quarters as quarter, i (i)}
			<QuarterPage {settings} {quarter} />
		{/each}
	{/if}
	{#if !settings.monthPage.disable && loadPages}
		{#each settings.months as month, i (i)}
			<MonthPage {settings} {month} />
		{/each}
	{/if}
	{#if !settings.weekPage.disable && loadPages}
		{#each settings.weeks as week, i (i)}
			<WeekPage {settings} {week} />
		{/each}
	{/if}
	{#if !settings.dayPage.disable && loadPages}
		{#each settings.days as day, i (i)}
			<DayPage {settings} {day} />
		{/each}
	{/if}
	{#if loadPages}
		{#each settings.collections as collection (collection.id)}
			<CollectionPages {settings} {collection} />
		{/each}
	{/if}
	{#if !settings.linksPage.disable && loadPages}
		{#each settings.linksPages as linkPage (linkPage.id)}
			<LinksPages {settings} {linkPage} />
		{/each}
	{/if}
</main>

<style lang="scss">
	main {
		font-family: var(--font);
		@supports (color: oklch(from var(--text) calc(l - 0.15) c h)) {
			--text-low: oklch(from var(--text) calc(l + 0.2) c h);
			--text-high: oklch(from var(--text) calc(l - 0.15) c h);
			--outline-low: oklch(from var(--outline) calc(l + 0.03) c h);
			--outline-high: oklch(from var(--outline) max(0, calc(l - 0.1)) c h);
		}
	}
	@media screen {
		main {
			overflow-y: auto;
			overflow-x: hidden;
			max-width: 100vw;
			max-height: 100vh;
		}
	}
	:global(main > article) {
		display: block;
		position: relative;
		background-color: var(--bg);
		width: var(--doc-width);
		height: var(--doc-height);
		content-visibility: auto;
		contain-intrinsic-size: 1px var(--doc-height);
	}
	@media print {
		:global(main.high-res > article) {
			transform: scale(2);
			transform-origin: top left;
		}
		:global(main.high-res > article:not(:nth-child(2))) {
			margin-top: calc(var(--doc-height) * 2);
		}
	}

	.menu-trigger {
		position: fixed;
		bottom: 1rem;
		right: 1rem;
		z-index: 10;
		background-color: var(--bg);
		color: currentColor;
		border-radius: 100%;
		width: 3.5rem;
		height: 3.5rem;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 1.35em;
		box-shadow: var(--shadow-4);
		cursor: pointer;
		&:hover {
			color: black;
		}
		@include tablet {
			right: 2rem;
		}
	}
	.menu {
		position: fixed;
		bottom: 5rem;
		right: 1rem;
		background-color: var(--bg);
		z-index: 10;
		width: 400px;
		max-width: calc(100vw - 2rem);
		max-height: 80vh;
		border-radius: var(--radius-5);
		box-shadow: var(--shadow-4);
		padding: 0 2rem 1rem;
		overflow-y: auto;
		overflow-x: hidden;
		@include tablet {
			right: 2rem;
		}

		@include scrollbar;
		&::-webkit-scrollbar-track-piece:start {
			margin-top: var(--radius-5);
		}
		&::-webkit-scrollbar-track-piece:end {
			margin-bottom: var(--radius-5);
		}
		h2 {
			position: sticky;
			top: 0;
			background-color: var(--bg);
			padding: 2rem 0 1rem;
			color: var(--text);
		}
		h3 {
			position: sticky;
			top: 4rem;
			background-color: var(--bg);
			color: var(--text);
			padding: 1rem 0;
			margin-top: 1rem;
			margin-bottom: -1rem;
		}
		.checkbox {
			margin: 0 0 0 0.5rem;
		}
	}
	form {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		margin: 0;
		fieldset {
			border: none;
			display: flex;
			flex-direction: column;
			padding: 0;
			label {
				font-size: 0.75rem;
				font-weight: 300;
				margin: 0 0 0.1rem 0.25rem;
			}
			input {
				width: 100%;
			}
		}
	}
	.actions {
		display: flex;
		flex-direction: column;
		gap: 1em;
		position: sticky;
		bottom: -1rem;
		background-color: white;
		width: 100%;
		padding: 1rem 0;
		.import-export {
			display: flex;
			justify-content: space-between;
			align-items: center;
			gap: 1rem;
		}
	}
	button.import-json,
	button.export-json,
	button.export {
		background-color: var(--action);
		color: var(--action-text);
		border-radius: 999px;
		padding: 0.75rem 1rem;
		font-size: 1.25rem;
		&:hover {
			background-color: var(--action-high);
			color: var(--action-text-high);
		}
	}

	button.export {
		width: 100%;
	}
	.collections {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		button {
			margin: 0.5rem 0 0;
		}
	}
	@media print {
		.menu,
		.menu-trigger {
			display: none;
		}
	}
</style>
