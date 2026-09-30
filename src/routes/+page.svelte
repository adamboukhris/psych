<!-- src/routes/+page.svelte -->
<script>
	import { run } from 'svelte/legacy';

	import { episodes } from '$lib/data.js';

	let searchQuery = $state('');
	let searchResults = $state([]);
	const seasons = Array.from({ length: 8 }, (_, index) => index + 1);

	// Reactive statement to handle filtering
	run(() => {
		const query = searchQuery.toLowerCase().trim();

		if (!query) {
			searchResults = [];
		} else {
			// Regex to detect if the user is searching for a specific season (e.g., "season 1")
			const seasonMatch = query.match(/season\s*(\d+)/);

			if (seasonMatch) {
				const seasonNum = parseInt(seasonMatch[1], 10);
				searchResults = episodes.filter((ep) => ep.season === seasonNum);
			} else {
				// Fallback to searching by series name or episode title
				searchResults = episodes.filter(
					(ep) => ep.series.toLowerCase().includes(query) || ep.title.toLowerCase().includes(query)
				);
			}
		}
	});
</script>

<main class="container">
	<header>
		<h1>Adam.Streaming</h1>
		<p class="subtitle">Search for a series, episode, or season (e.g., "Season 1")</p>
	</header>

	<div class="search-section">
		<div class="search-wrapper">
			<div class="glow-backdrop"></div>
			<input
				type="text"
				bind:value={searchQuery}
				placeholder="Search Psych, Season 1, Pilot..."
				class="search-input"
			/>
		</div>

		<div class="season-buttons" aria-label="Choose a season">
			{#each seasons as season}
				<button
					type="button"
					class:active={searchQuery.toLowerCase().trim() === `season ${season}`}
					onclick={() => (searchQuery = `Season ${season}`)}
				>
					Season {season}
				</button>
			{/each}
		</div>
	</div>

	{#if searchResults.length > 0}
		<div class="results-grid">
			{#each searchResults as episode (episode.id)}
				<div class="episode-card status-{episode.status}">
					<div class="card-header">
						<h3>{episode.title}</h3>
						<span class="badge">S{episode.season} E{episode.episodeNumber}</span>
					</div>
					<p class="series-name">{episode.series}</p>
					<a href={episode.url} class="play-link" target="_blank" rel="noopener noreferrer"
						>Play video</a
					>
				</div>
			{/each}
		</div>
	{:else if searchQuery && searchResults.length === 0}
		<p class="no-results">No episodes found for "{searchQuery}".</p>
	{/if}
</main>

<style>
	:global(body) {
		background-color: #0a0a0a;
		color: #ffffff;
		font-family:
			'Inter',
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			Roboto,
			Helvetica,
			Arial,
			sans-serif;
		margin: 0;
		padding: 0;
		-webkit-font-smoothing: antialiased;
	}

	.container {
		max-width: 900px;
		margin: 0 auto;
		padding: 4rem 2rem;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	header {
		text-align: center;
		margin-bottom: 3rem;
	}

	h1 {
		font-size: 2.5rem;
		font-weight: 700;
		letter-spacing: -0.05em;
		margin: 0 0 0.5rem 0;
	}

	.subtitle {
		color: #a3a3a3;
		font-size: 1rem;
		margin: 0;
	}

	/* Gemini-style animated glow search bar */
	.search-section {
		width: 100%;
		max-width: 600px;
		margin-bottom: 4rem;
	}

	.search-wrapper {
		position: relative;
		width: 100%;
	}

	.glow-backdrop {
		position: absolute;
		top: -4px;
		left: -4px;
		right: -4px;
		bottom: -4px;
		background: linear-gradient(90deg, #4285f4, #9b72cb, #d96570, #f4b400, #4285f4);
		background-size: 300% 300%;
		border-radius: 50px;
		filter: blur(12px);
		opacity: 0.6;
		z-index: -1;
		animation: glow-animation 6s ease infinite;
	}

	@keyframes glow-animation {
		0% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
		100% {
			background-position: 0% 50%;
		}
	}

	.search-input {
		width: 100%;
		padding: 1.25rem 2rem;
		font-size: 1.1rem;
		background-color: #111;
		color: #fff;
		border: 1px solid #333;
		border-radius: 50px;
		outline: none;
		box-sizing: border-box;
		transition: border-color 0.3s ease;
	}

	.search-input:focus {
		border-color: #555;
	}

	.search-input::placeholder {
		color: #666;
	}

	.season-buttons {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 0.6rem;
		margin-top: 1rem;
	}

	.season-buttons button {
		border: 1px solid #333;
		border-radius: 8px;
		padding: 0.55rem 0.85rem;
		background: #161616;
		color: #a3a3a3;
		cursor: pointer;
		font: inherit;
		font-size: 0.85rem;
		transition:
			background-color 0.2s ease,
			border-color 0.2s ease,
			color 0.2s ease;
	}

	.season-buttons button:hover,
	.season-buttons button.active {
		border-color: #4285f4;
		background: #1c2b46;
		color: #fff;
	}

	/* Results Grid & Cards matching image_6ca96e.jpg */
	.results-grid {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		width: 100%;
		max-width: 700px;
	}

	.episode-card {
		background-color: #161616; /* Same dark card background as the reference image */
		border-radius: 16px;
		padding: 1.5rem;
		transition:
			transform 0.2s ease,
			background-color 0.2s ease;
	}

	.episode-card:hover {
		background-color: #1c1c1c;
	}

	/* Dynamic Borders based on Status */
	.status-unwatched {
		border: 1px solid #333333; /* Grey */
	}

	.status-in-progress {
		border: 1px solid #d4b830; /* Yellow */
	}

	.status-finished {
		border: 1px solid #2ea043; /* Green matching the open to opportunities badge */
	}

	.card-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 0.5rem;
	}

	.card-header h3 {
		margin: 0;
		font-size: 1.1rem;
		font-weight: 600;
	}

	.badge {
		background-color: #222;
		padding: 0.25rem 0.75rem;
		border-radius: 20px;
		font-size: 0.8rem;
		color: #a3a3a3;
	}

	.series-name {
		color: #a3a3a3;
		font-size: 0.9rem;
		margin: 0 0 1rem 0;
	}

	.play-link {
		display: inline-block;
		color: #fff;
		text-decoration: none;
		font-size: 0.9rem;
		background-color: #333;
		padding: 0.5rem 1rem;
		border-radius: 8px;
		transition: background-color 0.2s ease;
	}

	.play-link:hover {
		background-color: #444;
	}

	.no-results {
		color: #a3a3a3;
		text-align: center;
		margin-top: 2rem;
	}

	/* Responsiveness */
	@media (max-width: 600px) {
		h1 {
			font-size: 2rem;
		}
		.search-input {
			padding: 1rem 1.5rem;
			font-size: 1rem;
		}
	}
</style>
