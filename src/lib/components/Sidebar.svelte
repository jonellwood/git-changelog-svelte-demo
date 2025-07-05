<script>
	export let changelogData;
	
	function formatDate(dateString) {
		const date = new Date(dateString);
		return date.toLocaleDateString('en-US', {
			year: 'numeric',
			month: 'short',
			day: 'numeric',
		});
	}
	
	function groupReleasesByMonth(releases) {
		const groups = {};
		releases.forEach(release => {
			const date = new Date(release.date);
			const monthKey = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`;
			const monthName = date.toLocaleDateString('en-US', {
				year: 'numeric',
				month: 'long',
			});
			
			if (!groups[monthKey]) {
				groups[monthKey] = {
					name: monthName,
					releases: []
				};
			}
			groups[monthKey].releases.push(release);
		});
		
		return Object.entries(groups).sort(([a], [b]) => b.localeCompare(a));
	}
	
	$: releasesByMonth = groupReleasesByMonth(changelogData.releases);
</script>

<aside class="changelog-sidebar">
	<div class="sidebar-header">
		<h2>📝 Changelog</h2>
		<p>Release History</p>
	</div>

	<div class="sidebar-content">
		<div class="sidebar-stats">
			<div class="stat">
				<span class="stat-number">{changelogData.stats.totalReleases}</span>
				<span class="stat-label">Releases</span>
			</div>
			<div class="stat">
				<span class="stat-number">{changelogData.stats.totalChanges}</span>
				<span class="stat-label">Changes</span>
			</div>
		</div>

		<div class="sidebar-releases">
			<div class="sidebar-section">
				<h3>📅 Releases by Month</h3>
				<div class="sidebar-summary">
					<span class="total-releases">{changelogData.releases.length} total releases</span>
				</div>
				<div class="release-groups">
					{#each releasesByMonth as [monthKey, monthData]}
						<details class="month-group" open={monthKey === new Date().toISOString().slice(0, 7)}>
							<summary class="month-summary">
								<span class="month-name">{monthData.name}</span>
								<span class="month-count">{monthData.releases.length} releases</span>
							</summary>
							<div class="month-releases">
								{#each monthData.releases as release}
									<a href="#{release.version}" class="release-link">
										<div class="release-version">{release.version}</div>
										<div class="release-date">{formatDate(release.date)}</div>
									</a>
								{/each}
							</div>
						</details>
					{/each}
				</div>
			</div>
		</div>
	</div>
</aside>
