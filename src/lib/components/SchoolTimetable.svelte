<script lang="ts">
	import type { PlannerSettings } from '$lib';

	let { settings = {} as PlannerSettings } = $props();
	const font = $derived(settings.design.font);
</script>

<div class="school-timetable">
	<img src="/school-table.jpg" alt="School Timetable" />
</div>

<style lang="scss">
	.school-timetable {
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;

		img {
			width: 100%;
			max-height: 100%;
		}
	}
</style>
