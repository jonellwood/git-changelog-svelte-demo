<script>
  import { writable, derived } from 'svelte/store';

  export let changelogData;
  
  // Initialize reactive variables
  let viewMode = 'grid';
  let searchTerm = '';
  let sortBy = 'date';
  let isLoading = false;
  let selectedRelease = null;
  let showModal = false;

  const releases = writable(changelogData.releases);
  const stats = writable(changelogData.stats);

  // Reactive statements for filtered and sorted releases
  $: filteredReleases = $releases.filter((release) => {
    if (!searchTerm) return true;
    const searchLower = searchTerm.toLowerCase();
    return (
      release.version.toLowerCase().includes(searchLower) ||
      release.summary.toLowerCase().includes(searchLower) ||
      release.rawContent.toLowerCase().includes(searchLower)
    );
  });

  $: sortedReleases = [...filteredReleases].sort((a, b) => {
    switch (sortBy) {
      case 'version':
        return versionCompare(b.version, a.version);
      case 'changes':
        return b.changeCount - a.changeCount;
      default:
        return new Date(b.date) - new Date(a.date);
    }
  });

  $: displayStats = {
    showing: filteredReleases.length,
    total: $releases.length,
    totalChanges: $stats.totalChanges || 0,
    latestVersion: $stats.latestVersion || 'Unknown',
  };

  function versionCompare(a, b) {
    const aParts = a.split('.').map(Number);
    const bParts = b.split('.').map(Number);

    for (let i = 0; i < Math.max(aParts.length, bParts.length); i++) {
      const aPart = aParts[i] || 0;
      const bPart = bParts[i] || 0;

      if (aPart !== bPart) {
        return aPart - bPart;
      }
    }
    return 0;
  }

  function formatDate(dateString) {
    const date = new Date(dateString);
    return date.toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'short',
      day: 'numeric',
    });
  }

  function getVersionIcon(version) {
    const parts = version.split('.');
    const major = parseInt(parts[0]) || 0;
    const minor = parseInt(parts[1]) || 0;
    const patch = parseInt(parts[2]) || 0;

    if (major > 0) return 'ph-rocket-launch';
    if (minor > 0) return 'ph-star';
    if (patch > 50) return 'ph-medal';
    return 'ph-tag';
  }

  function openReleaseModal(release) {
    selectedRelease = release;
    showModal = true;
  }

  function closeModal() {
    showModal = false;
    selectedRelease = null;
  }

  function formatMarkdown(content) {
    if (!content) return '';

    // Simple markdown-like formatting
    return content
      .split('\n')
      .map((line) => {
        if (line.trim().startsWith('- ')) {
          return `<li>${line.trim().substring(2)}</li>`;
        }
        return line;
      })
      .join('\n')
      .replace(/(\<li\>.*\<\/li\>\n?)+/g, (match) => `<ul>${match}</ul>`);
  }
</script>

<div class="changelog-container">
  <!-- View Toggle Buttons -->
  <div class="view-controls">
    <div class="view-toggles">
      <button 
        class="toggle-btn" 
        class:active={viewMode === 'grid'}
        on:click={() => viewMode = 'grid'}
      >
        <i class="ph-duotone ph-squares-four"></i>
        Grid
      </button>
      <button 
        class="toggle-btn" 
        class:active={viewMode === 'list'}
        on:click={() => viewMode = 'list'}
      >
        <i class="ph-duotone ph-list"></i>
        List
      </button>
    </div>
  </div>

  <!-- Search and Filter Bar -->
  <div class="filter-bar">
    <div class="filter-left">
      <div class="search-box">
        <i class="ph-duotone ph-magnifying-glass"></i>
        <input 
          type="text" 
          bind:value={searchTerm} 
          placeholder="Search releases..."
          class="search-input"
        />
      </div>
    </div>
    <div class="filter-right">
      <div class="sort-dropdown">
        <label>Sort by:</label>
        <select bind:value={sortBy}>
          <option value="date">Date</option>
          <option value="version">Version</option>
          <option value="changes">Changes</option>
        </select>
      </div>
      <div class="results-count">
        Showing {displayStats.showing} of {displayStats.total} releases
      </div>
    </div>
  </div>

  <!-- Quick Stats -->
  <div class="stats-bar">
    <div class="stat-card">
      <div class="stat-icon">
        <i class="ph-duotone ph-chart-line"></i>
      </div>
      <div class="stat-content">
        <div class="stat-number">{displayStats.total}</div>
        <div class="stat-label">Total Releases</div>
      </div>
    </div>
    <div class="stat-card">
      <div class="stat-icon">
        <i class="ph-duotone ph-list-bullets"></i>
      </div>
      <div class="stat-content">
        <div class="stat-number">{displayStats.totalChanges}</div>
        <div class="stat-label">Total Changes</div>
      </div>
    </div>
    <div class="stat-card">
      <div class="stat-icon">
        <i class="ph-duotone ph-rocket-launch"></i>
      </div>
      <div class="stat-content">
        <div class="stat-number">{displayStats.latestVersion}</div>
        <div class="stat-label">Latest Version</div>
      </div>
    </div>
  </div>

  <!-- Loading State -->
  {#if isLoading}
    <div class="loading-state">
      <div class="loading-spinner"></div>
      <p>Loading releases...</p>
    </div>
  <!-- Empty State -->
  {:else if sortedReleases.length === 0}
    <div class="empty-state">
      <div class="empty-icon">
        <i class="ph-duotone ph-file-text"></i>
      </div>
      <h3>No releases found</h3>
      {#if searchTerm}
        <p>Try adjusting your search criteria</p>
      {:else}
        <p>No releases have been published yet</p>
      {/if}
    </div>
  <!-- Releases Grid -->
  {:else if viewMode === 'grid'}
    <div class="releases-grid">
      {#each sortedReleases as release}
        <div 
          class="release-card"
          id={release.version}
          on:click={() => openReleaseModal(release)}
          role="button"
          tabindex="0"
          on:keydown={(e) => e.key === 'Enter' && openReleaseModal(release)}
        >
          <div class="release-header">
            <div class="release-icon">
              <i class={getVersionIcon(release.version)}></i>
            </div>
            <div class="release-info">
              <h3 class="release-version">{release.version}</h3>
              <p class="release-date">{formatDate(release.date)}</p>
            </div>
            <div class="release-tag">
              <span class="tag-label">{release.tag}</span>
            </div>
          </div>
          
          <div class="release-body">
            <div class="release-summary">
              <p>{release.summary}</p>
            </div>
          </div>
          
          <div class="release-footer">
            <div class="release-stats">
              <span class="stat-item">
                <i class="ph-duotone ph-list-bullets"></i>
                {release.changeCount} changes
              </span>
              <span class="stat-item">
                <i class="ph-duotone ph-calendar"></i>
                {formatDate(release.date)}
              </span>
            </div>
            <div class="release-actions">
              <button class="action-btn primary" on:click={() => openReleaseModal(release)}>
                <i class="ph-duotone ph-arrow-right"></i>
                View Release
              </button>
            </div>
          </div>
        </div>
      {/each}
    </div>
  <!-- Releases List -->
  {:else if viewMode === 'list'}
    <div class="releases-list">
      {#each sortedReleases as release}
        <div 
          class="release-row"
          id={release.version}
          on:click={() => openReleaseModal(release)}
          role="button"
          tabindex="0"
          on:keydown={(e) => e.key === 'Enter' && openReleaseModal(release)}
        >
          <div class="release-list-header">
            <div class="release-icon">
              <i class={getVersionIcon(release.version)}></i>
            </div>
            <div class="release-info">
              <h3 class="release-version">{release.version}</h3>
              <p class="release-summary">{release.summary}</p>
            </div>
            <div class="release-meta">
              <span class="release-date">{formatDate(release.date)}</span>
              <span class="release-changes">{release.changeCount} changes</span>
              <span class="release-tag">{release.tag}</span>
            </div>
            <div class="release-actions">
              <button class="action-btn primary" on:click={() => openReleaseModal(release)}>
                <i class="ph-duotone ph-arrow-right"></i>
                View
              </button>
            </div>
          </div>
        </div>
      {/each}
    </div>
  {/if}

  <!-- Release Modal -->
  {#if showModal}
    <div class="modal-overlay" on:click={closeModal} role="button" tabindex="0" on:keydown={(e) => e.key === 'Escape' && closeModal()}>
      <div class="modal-content" on:click|stopPropagation role="dialog">
        <div class="modal-header">
          <h2 class="modal-title">{selectedRelease?.version}</h2>
          <button class="modal-close" on:click={closeModal}>
            <i class="ph-duotone ph-x"></i>
          </button>
        </div>
        <div class="modal-body">
          <div class="release-detail">
            <div class="release-meta">
              <div class="meta-item">
                <i class="ph-duotone ph-calendar"></i>
                {formatDate(selectedRelease?.date)}
              </div>
              <div class="meta-item">
                <i class="ph-duotone ph-list-bullets"></i>
                {selectedRelease?.changeCount} changes
              </div>
              <div class="meta-item">
                <i class="ph-duotone ph-tag"></i>
                {selectedRelease?.tag}
              </div>
            </div>
            <div class="release-content">
              <div class="release-summary">
                <p>{selectedRelease?.summary}</p>
              </div>
              <div class="markdown-content">
                {@html formatMarkdown(selectedRelease?.rawContent)}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  {/if}
</div>
