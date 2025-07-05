<script>
	import { onMount } from 'svelte';
	import { writable, derived } from 'svelte/store';
	import ChangelogManager from '$lib/components/ChangelogManager.svelte';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import Header from '$lib/components/Header.svelte';
	import { changelogData } from '$lib/data/sampleData.js';

	let appName = 'SvelteKit Changelog Renderer';

	onMount(async () => {
		try {
			const response = await fetch('/package.json');
			if (response.ok) {
				const packageData = await response.json();
				appName = packageData.name
					.split('-')
					.map(word => word.charAt(0).toUpperCase() + word.slice(1))
					.join(' ') + ' Release Notes';
			}
		} catch (error) {
			console.warn('Could not load package.json:', error);
		}
	});
</script>

<div class="changelog-layout">
	<!-- Sidebar Navigation -->
	<Sidebar {changelogData} />

	<!-- Main Content -->
	<main class="changelog-main">
		<!-- Page Header -->
		<Header {appName} />

		<!-- Changelog Manager Component -->
		<ChangelogManager {changelogData} />
	</main>
</div>
