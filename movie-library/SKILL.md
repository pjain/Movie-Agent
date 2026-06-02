---
name: movie-agent
description: international movie and television discovery plus personal watch tracking for OpenClaw, Hermes, and compatible agents. Use when the user asks to find films, TV shows, seasons, episodes, IMDb or Rotten Tomatoes ratings, where to watch by geography, add titles to a watchlist, mark movies or shows watched, rate titles with thumbs up/down or five-star ratings, remove titles from lists, recommend similar titles, or find work by director, actor, creator, network, streamer, studio, production house, genre, country, language, or franchise.
---

# Movie Agent

## Purpose

Use this skill to manage international movie and television discovery, ratings, streaming availability, watchlists, watched history, and recommendations. The skill is instruction-first: use the host agent's existing web, API, file-read, and file-write tools. Do not require bundled scripts for normal use.

This skill supports:

- Movies, short films, documentaries, and specials.
- TV shows, miniseries, anime, reality TV, web series, seasons, and episodes.
- International titles, alternate titles, original-language titles, country-of-origin filters, and geography-specific streaming availability.
- User ratings using either thumbs up/down or a normalized 0.5-5.0 star scale.

## First-run setup

On first use, check whether a local data file exists at the configured path. Default path: `~/.movie-agent/library.json`.

If no profile exists, ask:

> What geography should I use by default for streaming availability? You can also tell me later when you are temporarily in another geography.

Store the answer as `profile.default_region` using a stable country/region code when possible, such as `US`, `IN`, `GB`, `CA`, `JP`, `KR`, or `FR`.

If the user says they are temporarily in another geography, use that region for the current lookup only unless they explicitly ask to change their default.

## Core entity model

Use the neutral term **title** for both movies and television.

Every stored title must include `media_type` where possible:

- `movie`
- `tv`
- `season`
- `episode`
- `special`
- `unknown`

For TV, distinguish show-level, season-level, and episode-level actions:

- "Add Shogun" usually means add the show.
- "I watched Shogun season 1" means mark season 1 watched.
- "I watched episode 3 of Shogun" means mark that episode watched when episode tracking is supported.
- If the level is ambiguous and the action affects storage, ask a concise clarification.

## Data source preferences

Use the best available provider in this order. If a preferred source is unavailable, use the next reliable source and briefly disclose the limitation.

1. **Canonical metadata for movies and TV:** TMDB or another structured movie/TV database. Capture stable IDs, original title/name, release/first-air year, country, original language, genres, cast, crew, creators, networks, production companies, seasons, and episodes.
2. **IMDb and Rotten Tomatoes ratings:** OMDb when available, because it can return IMDb ratings and Rotten Tomatoes ratings in one response when matched by IMDb ID. Otherwise use a reliable structured source or current web source.
3. **Where to watch by geography:** Watchmode, JustWatch-compatible data, TMDB watch providers, or another current streaming availability API. Availability changes often; always perform a fresh lookup unless the user explicitly asks to use cached data.
4. **International enrichment:** Use original-language title, alternate titles, country of origin, release/air dates by region, and local provider names when available.
5. **Recommendations:** Combine structured similar/recommendation endpoints, genre/person/company/network/country/language metadata, the local watched list, watchlist exclusions, and the user's stated preferences.

Never fabricate ratings, availability, seasons, episodes, or provider support. Use `unknown` when a source does not provide the field.

## Local storage rules

Maintain a local JSON file at `~/.movie-agent/library.json` unless the user or runtime config specifies a different path.

- Create the file if it does not exist.
- Preserve existing entries and unknown fields.
- Normalize titles with media type, year, stable external IDs, original language, and country where available.
- De-duplicate by stable ID first, then `media_type + normalized title/name + year`.
- Use ISO dates such as `2026-06-01` for `added_at`, `watched_at`, `rated_at`, and `updated_at`.
- Read the existing file before writing. Avoid destructive overwrites.
- If filesystem access is unavailable, maintain a response-local representation and state that persistence is unavailable.

Consult `references/schemas.md` before writing records.

## Rating system

Users may rate movies, shows, seasons, or episodes with thumbs up/down, explicit stars, or natural language.

Store both raw input and normalized values:

- `rating.raw`: original user phrase, such as `I loved it`, `thumbs up`, or `4/5`.
- `rating.scale`: `stars_5`, `thumbs`, or `inferred_stars_5`.
- `rating.stars`: number from 0.5 to 5.0 when available.
- `rating.thumbs`: `up`, `down`, or `neutral` when available.
- `rating.sentiment`: `positive`, `mixed`, `negative`, or `neutral`.

Interpret explicit ratings directly:

- `5 stars`, `5/5`, `five stars` -> 5.0 stars.
- `4 stars`, `4/5` -> 4.0 stars.
- `3.5 stars`, `3.5/5` -> 3.5 stars.
- `thumbs up` -> thumbs `up`; if a star value is needed, treat as 4.0.
- `thumbs down` -> thumbs `down`; if a star value is needed, treat as 2.0.

Interpret these natural-language ratings when the phrase clearly refers to the title being watched/rated:

- `I loved it`, `loved it`, `love it` -> 5.0 stars, positive.
- `Fantastic` -> 4.0 stars, positive.
- `Good` -> 3.5 stars, positive/mixed.
- `Ok`, `Okay` -> 3.0 stars, mixed/neutral.

Do not over-infer vague reactions. If the phrase is ambiguous, store `rating.raw` and ask for clarification only if a precise rating is required for the current task.

Consult `references/rating-guide.md` for more examples.

## Task workflows

### Find title

Input may be one title or multiple titles. For each title:

1. Resolve whether the result is a movie, TV show, season, episode, or ambiguous title.
2. Prefer exact stable ID matches if the user provides an IMDb/TMDB/TVDB ID.
3. If multiple plausible matches exist, use media type, year, country, language, cast, or user context. If still ambiguous, show the top few candidates with media type, year, country/language, and creator/director.
4. Retrieve canonical metadata, IMDb rating, Rotten Tomatoes rating when available, and geography-specific watch providers.
5. For TV, include status, season count, episode count, original network/streamer, first-air year, and latest/next season or episode when available.
6. Return a concise table or structured list.

Default output fields:

- Title/name and media type
- Year or first-air year
- Country/language when useful
- IMDb rating
- Rotten Tomatoes rating when available
- Where to watch in selected geography
- Notes on ambiguity or missing data

### Add to watchlist

When the user asks to add a movie or show:

1. Resolve the title and media type.
2. Fetch basic metadata if missing.
3. If it already exists in `watched`, ask whether to also keep it on the watchlist or skip adding.
4. If it already exists in `watchlist`, update metadata and `updated_at`; do not duplicate.
5. Store show-level TV records by default unless the user specifies season or episode.
6. Save the JSON file and confirm with title/name, year, and media type.

### Mark watched

When the user says they watched a movie, show, season, or episode:

1. Resolve the title against `watchlist`, `watched`, and fresh lookup results.
2. Determine watched level: movie, show, season, or episode.
3. Add or update the record in `watched` with `status: "watched"` and `watched_at`.
4. Move matching watchlist items out of `watchlist` unless the user asks to keep them there.
5. Capture optional rating, notes, rewatch flag, watched geography, and watched platform if provided.
6. Save the JSON file and confirm.

Examples:

- "I watched Parasite and loved it" -> mark movie watched and infer 5.0 stars.
- "Finished Shogun season 1, fantastic" -> mark season 1 watched and infer 4.0 stars.
- "Episode 3 was ok" -> if context identifies the show/season, mark episode 3 watched and infer 3.0 stars.

### Rate title

When the user rates something without saying watched:

1. Resolve the title and media type.
2. Normalize the rating using the rating rules.
3. If the title is already in `watched`, update that record.
4. If it is only in `watchlist`, update that record unless the phrase implies the user watched it; ask only if necessary.
5. If it is not in any list, create a minimal record under `watched` if the wording implies viewing, otherwise create/update a `ratings` list if supported by the current schema.

### Remove

When the user asks to remove a title:

1. Search all local lists: `watchlist`, `watched`, `ratings`, and auxiliary lists.
2. If there is one clear match, remove it from all active lists.
3. If there are multiple matches across movie/TV or seasons/episodes, ask which one before deleting.
4. Confirm exactly what was removed.

### Recommend

For recommendations:

1. Load watched history, user ratings, watchlist, preferences, and default geography.
2. Prefer highly rated watched titles as positive signals. Treat thumbs down or low ratings as negative signals.
3. Support filters by media type, genre, country, language, actor, director, creator, network, streamer, production house, franchise, decade, and geography.
4. Exclude titles already watched unless the user asks for rewatch ideas.
5. De-prioritize watchlist items unless the user asks what to watch next from the watchlist.
6. Annotate current streaming availability in the selected geography.
7. Return 5-10 recommendations with one-sentence rationale each.

### Find by person, company, network, country, language, genre, or franchise

Support requests like:

- "Find shows created by Phoebe Waller-Bridge."
- "Find Korean thrillers available in the US."
- "Find all movies by Studio Ghibli."
- "Find HBO miniseries I haven't watched."
- "Find films starring Tabu."

Workflow:

1. Identify entity type: director, actor, creator, writer, showrunner, genre, production company, network, streamer, country, language, or franchise.
2. Use a structured source such as TMDB when available.
3. Distinguish acted-in, directed-by, created-by, written-by, network, and production-company relationships.
4. Sort results by the user's request. Default to relevance/popularity for broad queries.
5. Annotate watched/watchlist/rating status from local storage.
6. If requested, filter by geography-specific streaming availability.

## V2 integrations: Notion and Airtable

For v1, store lists locally in JSON only.

If the user asks to use Notion or Airtable:

1. Explain that this requires authorization and workspace/database/table details.
2. Ask for the appropriate authorization/setup path supported by the runtime. Do not request secrets in an insecure chat surface if the platform has secure setup or environment-variable support.
3. Do not migrate or sync until the user explicitly authorizes the integration and confirms the destination.
4. Continue to keep a local JSON backup unless the user asks not to.

## Safety and reliability

- Do not scrape sites that prohibit automated access when a structured API is available.
- Do not guess ratings, providers, TV seasons, episodes, release dates, or availability.
- Treat streaming availability as time-sensitive and refresh it for each lookup.
- Keep API keys out of chat transcripts when the runtime supports secure configuration.
- Confirm ambiguous destructive operations.
- Preserve user privacy: do not sync watch history to third-party services without explicit authorization.

## Reference files

- `references/schemas.md`: local JSON schema, title record shape, TV season/episode tracking, rating fields, and output templates.
- `references/provider-guide.md`: recommended providers, fields to request, international TV handling, and fallback behavior.
- `references/rating-guide.md`: rating normalization and natural-language interpretation rules.
