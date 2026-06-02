# Provider Guide

## Recommended providers

### TMDB

Use for movies and TV:

- Title search and disambiguation across movie and TV media types.
- Stable TMDB IDs and external IDs such as IMDb ID.
- Directors, creators, writers, showrunners, cast, genres, runtime, episode runtime, networks, production companies, countries, languages, seasons, and episodes.
- Similar/recommended movies and shows.
- Watch provider links where available.

Recommended lookup pattern:

1. Search multi-title when media type is ambiguous.
2. Fetch movie or TV details after selecting a candidate.
3. Include credits, external IDs, alternative titles, translations, release dates/content ratings, recommendations, similar titles, and watch providers when supported by the host API tool.
4. For TV, fetch season or episode details only when the user asks for season/episode-level tracking or ratings.

### OMDb

Use for:

- IMDb rating.
- Rotten Tomatoes rating when included in the `Ratings` array.
- Basic fallback metadata for movies and some series.

Match carefully using IMDb ID when possible. Title-only searches can return the wrong remake, sequel, regional title, or TV/movie variant.

### Watchmode, JustWatch-compatible providers, TMDB watch providers, or equivalent

Use for:

- Current streaming availability by country/region.
- Access type: subscription, rent, buy, free, ads, unavailable.
- Movie and TV availability, including show-level availability where season/episode-level availability is unavailable.
- Deep links when available.

Availability changes often. Refresh availability for every direct user request about where to watch.

### TVDB, Trakt, Wikidata, or official network/streamer sources

Use as fallback or enrichment for:

- International television metadata.
- Episode lists and air dates.
- Alternate titles and original-language names.
- Networks, streamers, and region-specific release details.

Prefer structured APIs over scraping.

## International handling

Capture these fields when available:

- `original_title` or `original_name`.
- `original_language`.
- `spoken_languages`.
- `countries` or countries of origin.
- Localized title for the user's geography.
- Region-specific release date or first-air date.
- Region-specific providers and access types.

For international titles:

- Preserve the user's entered title in notes or `aliases` when it differs from the canonical title.
- Prefer original title plus localized title when both are useful.
- Avoid assuming country from language alone.
- For anime and non-English shows, distinguish original network/studio from international streaming distributor.

## Fallback behavior

If a source is unavailable:

1. Use another structured provider before generic web search.
2. If generic web search is required, prefer official provider pages, movie/TV database pages, network/streamer pages, or reputable entertainment databases.
3. Mark missing fields as `unknown`; do not infer ratings, episodes, seasons, or availability.
4. Mention which fields could not be verified.

## Geography handling

- Store `profile.default_region` after first-run setup.
- Accept natural language regions such as `India`, `US`, `UK`, `Canada`, `Japan`, `South Korea`, and normalize when possible.
- If the user says `I'm traveling in India this week`, use `IN` for the current query only unless they ask to change the default.
- When provider results are returned by country code, pass the normalized region code to the API/tool.

## Common user intents

- `find movie X` -> movie metadata + ratings + where to watch.
- `find show X` -> TV metadata + ratings + where to watch.
- `add X` -> resolve title/media type, store in watchlist.
- `I watched X` -> move from watchlist to watched.
- `I loved X` -> rate as 5.0 stars; mark watched if wording implies viewing.
- `X was fantastic` -> rate as 4.0 stars.
- `X was good` -> rate as 3.5 stars.
- `X was ok` -> rate as 3.0 stars.
- `thumbs up for X` -> store thumbs up; infer 4.0 only if needed.
- `remove X` -> remove from all local lists after ambiguity check.
- `recommend shows like X` -> similar/recommendations + user history + availability.
- `find all shows by X` -> determine whether X is a creator, actor, writer, network, production company, genre, country, or language.
