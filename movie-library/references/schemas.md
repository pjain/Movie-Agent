# Movie Agent Schemas

## Local data file

Default path: `~/.movie-agent/library.json`

```json
{
  "schema_version": "2.0",
  "profile": {
    "default_region": null,
    "preferred_platforms": [],
    "preferred_genres": [],
    "preferred_languages": [],
    "preferred_countries": [],
    "favorite_titles": [],
    "favorite_people": [],
    "updated_at": null
  },
  "watchlist": [],
  "watched": [],
  "ratings": [],
  "removed": []
}
```

## Title record

Use this shape for movies, shows, seasons, and episodes. Omit fields that are not applicable, but preserve unknown existing fields.

```json
{
  "title": "Shogun",
  "original_title": "Shogun",
  "media_type": "tv",
  "year": 2024,
  "status": "watchlist",
  "level": "show",
  "season_number": null,
  "episode_number": null,
  "ids": {
    "tmdb": "126308",
    "imdb": "tt2788316",
    "tvdb": null
  },
  "ratings_external": {
    "imdb": "8.6",
    "rotten_tomatoes": "99%"
  },
  "rating_user": {
    "raw": "Fantastic",
    "scale": "inferred_stars_5",
    "stars": 4.0,
    "thumbs": "up",
    "sentiment": "positive",
    "confidence": "high",
    "rated_at": "2026-06-01"
  },
  "genres": ["Drama", "War & Politics"],
  "countries": ["US", "JP"],
  "original_language": "en",
  "spoken_languages": ["en", "ja"],
  "directors": [],
  "creators": ["Rachel Kondo", "Justin Marks"],
  "cast": ["Hiroyuki Sanada", "Cosmo Jarvis", "Anna Sawai"],
  "networks": ["FX"],
  "production_companies": ["FX Productions"],
  "runtime_minutes": null,
  "episode_runtime_minutes": [59],
  "number_of_seasons": 1,
  "number_of_episodes": 10,
  "first_air_date": "2024-02-27",
  "last_air_date": "2024-04-23",
  "where_to_watch": {
    "region": "US",
    "checked_at": "2026-06-01",
    "providers": [
      {
        "platform": "Hulu",
        "access_type": "subscription",
        "deep_link": null
      }
    ]
  },
  "added_at": "2026-06-01",
  "watched_at": null,
  "notes": "",
  "updated_at": "2026-06-01"
}
```

## Media type and level values

`media_type` values:

- `movie`
- `tv`
- `season`
- `episode`
- `special`
- `unknown`

`level` values:

- `title`
- `show`
- `season`
- `episode`

Examples:

- Movie: `media_type: "movie"`, `level: "title"`.
- TV show: `media_type: "tv"`, `level: "show"`.
- TV season: `media_type: "season"`, `level: "season"`, `season_number: 1`.
- TV episode: `media_type: "episode"`, `level: "episode"`, `season_number: 1`, `episode_number: 3`.

## User rating schema

```json
{
  "raw": "I loved it",
  "scale": "inferred_stars_5",
  "stars": 5.0,
  "thumbs": "up",
  "sentiment": "positive",
  "confidence": "high",
  "rated_at": "2026-06-01"
}
```

`scale` values:

- `stars_5`
- `thumbs`
- `inferred_stars_5`
- `unknown`

`thumbs` values:

- `up`
- `down`
- `neutral`
- `unknown`

`sentiment` values:

- `positive`
- `mixed`
- `negative`
- `neutral`
- `unknown`

## De-duplication

Use this order:

1. Match by `ids.imdb` if available.
2. Match by `ids.tmdb` plus `media_type` if available.
3. Match by `ids.tvdb` for TV if available.
4. Match by normalized `media_type + title/original_title + year`.
5. For seasons and episodes, include `season_number` and `episode_number`.
6. If media type or year is missing and more than one plausible match exists, treat as ambiguous.

## Status values

- `watchlist`: user wants to watch.
- `watched`: user has watched.
- `rated`: user has rated but watch status is unclear.
- `removed`: optional tombstone only.

## Find title output template

```markdown
| Title | Type | Year | IMDb | Rotten Tomatoes | Where to watch in {REGION} | Notes |
|---|---|---:|---:|---:|---|---|
| Shogun | TV show | 2024 | 8.6 | 99% | Hulu subscription | Availability checked today |
```

## Recommendation output template

```markdown
1. **Pachinko (TV, 2022-)** - International family drama with historical scope. Available on: {providers}. IMDb: {rating}; RT: {score}.
```
