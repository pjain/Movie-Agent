# Movie and TV Show Library Skill

A reusable movie and TV show discovery and personal watch-tracking skill for Hermes, OpenClaw, and compatible agent runtimes.

The skill helps an agent find movie ratings, check where movies stream in the user's geography, maintain a local watchlist and watched list, remove movies from all lists, recommend similar movies, and find films by director, actor, genre, or production house.

## Capabilities

### Find a Movie or TV Show

Input: one movie/tv show title or multiple titles.

Output includes:

- Canonical movie/tv show title and year
- IMDb rating
- Rotten Tomatoes rating when available
- Streaming availability by geography
- Access type such as subscription, rent, buy, free, ads, or unavailable
- Source/provider notes when data is missing or ambiguous

Example:

```text
Find The Dark Knight and tell me IMDb, Rotten Tomatoes, and where I can watch it in the US.
```

### Add Movie/TV show

Adds a movie to the local watchlist with normalized metadata, stable IDs when available, ratings, streaming availability, and timestamps.

```text
Add Dune: Part Two to my watchlist.
```

```text
Add Star Wars Andor to my watchlist.
```

### Watched

Moves a movie/show from the watchlist to the watched list, or adds it directly to watched history if it was not already in the watchlist.

```text
Mark Arrival as watched. I watched it yesterday and would rate it 5/5.
```
or
```text
I watched the first 3 episodes of Seal Team season 1 and enjoyed it.
```

### Remove

Removes a matching movie from all active lists. The skill may preserve a minimal tombstone in `removed` so the agent can avoid accidental re-adds or explain history later.

```text
Remove Passengers (2016) from all lists.
```

### Recommend

Recommends movies/shows similar to a movie/show, genre, director, actor, production house, or based on watched history and preferences. Recommendations exclude already watched movies unless the user asks otherwise.

```text
Recommend movies like Arrival that I have not watched.
Recommend A24 horror movies available to stream in Germany.
Recommend sci-fi films based on my watched history.
Recommend shows like Game of Thrones that I haven't seen yet.
Recommend comedy shows for the family.
```

### Find By

Finds movies by director, actor, genre, or production house.

```text
Find movies directed by Denis Villeneuve.
Find movies starring Al Pacino.
Find movies produced by A24.
Find shows starring Jason Sudeikis.
Find dramatic shows available on Hulu in the US.
```

## Runtime assumptions

This is an instruction-first skill. It does **not** bundle API-calling scripts. Instead, it tells the agent how to use whatever web, API, file-read, and file-write tools the runtime already provides.

Recommended external data sources:

- **TMDB** for canonical movie metadata, IDs, cast, crew, genres, production companies, similar movies, and recommendations.
- **OMDb** for IMDb ratings and Rotten Tomatoes ratings when available.
- **Watchmode, JustWatch-compatible providers, TMDB watch providers, or equivalent current availability sources** for streaming platforms by geography.

The agent should not fabricate ratings or streaming availability. If a provider does not return a value, the skill instructs the agent to return `unknown` or briefly explain the limitation.

## Geography behavior

On first use, the skill asks the user for their preferred geography for streaming availability.

```text
What geography should I use by default for streaming availability? You can also tell me later when you are temporarily in another geography.
```

The skill stores the answer as a stable country/region code where possible, such as `US`, `IN`, or `GB`.

Temporary geography changes are supported:

```text
I am in India this week. Where can I watch Oppenheimer?
```

In that case, the agent uses the temporary region for the current lookup only unless the user explicitly asks to change the default.

## Local storage

Default local data path:

```text
~/.movie-library/movies.json
```

Default shape:

```json
{
  "profile": {
    "default_region": null,
    "preferred_platforms": [],
    "preferred_genres": [],
    "favorite_movies": [],
    "favorite_people": [],
    "updated_at": null
  },
  "watchlist": [],
  "watched": [],
  "removed": []
}
```

See [`movie-library/references/schemas.md`](movie-library/references/schemas.md) for the full schema and mutation rules.

## Repository layout

```text
movie-library/
  SKILL.md
  agents/
    openai.yaml
  references/
    provider-guide.md
    schemas.md
README.md
.gitignore
LICENSE
```

## Installing in Hermes

Hermes skills live in `~/.hermes/skills/` by default and use `SKILL.md` as the main entrypoint. Clone this repository, then place the skill directory under Hermes skills:

```bash
mkdir -p ~/.hermes/skills
cp -R movie-library ~/.hermes/skills/movie-library
```

Then use it from Hermes:

```text
/movie-library find The Dark Knight
/movie-library add Dune: Part Two to my watchlist
/movie-library mark Arrival as watched
```

Hermes also supports external skill directories via `skills.external_dirs` in `~/.hermes/config.yaml`:

```yaml
skills:
  external_dirs:
    - ~/repos/movie-library
```

## Installing in OpenClaw or compatible runtimes

Install the `movie-library/` folder wherever your runtime scans skill directories. The runtime must expose enough capabilities for the agent to:

1. Read and write the local JSON file.
2. Search/call web or API providers for current movie metadata, ratings, and streaming availability.
3. Load `SKILL.md` and reference files on demand.

## Configuration

No API keys are bundled in this skill.

Suggested optional environment variables for runtimes that support them:

```bash
TMDB_API_KEY=...
OMDB_API_KEY=...
WATCHMODE_API_KEY=...
```

The skill itself does not require these names. They are recommended conventions for the host agent/tooling layer.

## Example output

```json
{
  "title": "The Dark Knight",
  "year": 2008,
  "imdb_rating": "9.0",
  "rotten_tomatoes_rating": "94%",
  "where_to_watch": [
    {
      "platform": "Max",
      "region": "US",
      "access_type": "subscription",
      "link": null
    }
  ],
  "notes": []
}
```

## V2 extension points

Future versions can add structured persistence providers:

- Notion database sync
- Airtable base sync
- SQLite local database
- Multi-user profiles
- Platform preference weighting
- Better recommendation scoring from watch history

For Notion or Airtable, the skill should ask the user for explicit authorization and destination details before reading or writing anything.

## Security and privacy

- Do not store API keys inside `movies.json`.
- Do not write private movie history to Notion, Airtable, or any third-party destination unless the user explicitly authorizes it.
- Do not fabricate ratings, streaming availability, or cast/crew data.
- Prefer stable IDs such as IMDb ID and TMDB ID to avoid mixing remakes, sequels, and regional releases.
- Confirm ambiguous destructive operations, especially removals involving multiple possible title matches.

## Development notes

This repo intentionally avoids bundled executable scripts in v1. The goal is to keep the skill portable across Hermes, OpenClaw, and other agent runtimes with different tool layers.

If scripts are added later, keep them deterministic, documented, and optional. The main skill should continue to work as an instruction-only workflow when the host agent already has web/API/file tools.
