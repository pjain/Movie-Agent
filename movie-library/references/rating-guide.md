# Rating Guide

## Supported rating inputs

Users may rate movies, TV shows, seasons, or episodes using:

- Explicit five-star ratings: `5 stars`, `4/5`, `3.5 stars`.
- Thumbs: `thumbs up`, `thumbs down`.
- Natural language reactions: `I loved it`, `Fantastic`, `Good`, `Ok`.

Always preserve the original user wording in `rating.raw`.

## Normalized rating object

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

## Explicit star mapping

| User input | Normalized stars | Scale | Notes |
|---|---:|---|---|
| `5 stars`, `5/5`, `five stars` | 5.0 | `stars_5` | Direct rating |
| `4.5 stars`, `4.5/5` | 4.5 | `stars_5` | Direct rating |
| `4 stars`, `4/5` | 4.0 | `stars_5` | Direct rating |
| `3.5 stars`, `3.5/5` | 3.5 | `stars_5` | Direct rating |
| `3 stars`, `3/5` | 3.0 | `stars_5` | Direct rating |
| `2 stars`, `2/5` | 2.0 | `stars_5` | Direct rating |
| `1 star`, `1/5` | 1.0 | `stars_5` | Direct rating |

Clamp five-star values to the 0.5-5.0 range unless the user explicitly uses a different scale. If the user says `8/10`, either store `raw` and convert to 4.0 stars only if conversion is useful, or ask which scale they prefer.

## Thumbs mapping

| User input | Thumbs | Stars if needed | Sentiment |
|---|---|---:|---|
| `thumbs up`, `liked it` | `up` | 4.0 | positive |
| `thumbs down`, `didn't like it` | `down` | 2.0 | negative |
| `meh`, `mixed` | `neutral` | 3.0 | mixed |

When the user only gives thumbs, store `scale: "thumbs"`. Only infer stars when recommendations, ranking, or compatibility with a star-only storage format requires it.

## Required natural-language mappings

| User phrase | Stars | Thumbs | Sentiment | Confidence |
|---|---:|---|---|---|
| `I loved it`, `loved it`, `love it` | 5.0 | up | positive | high |
| `Fantastic` | 4.0 | up | positive | high |
| `Good` | 3.5 | up | positive/mixed | medium |
| `Ok`, `Okay` | 3.0 | neutral | mixed/neutral | medium |

These mappings apply when the phrase clearly refers to the title being watched or rated.

## Additional suggested mappings

These are optional defaults; do not over-infer if the context is unclear.

| User phrase | Stars | Sentiment |
|---|---:|---|
| `masterpiece`, `perfect`, `amazing` | 5.0 | positive |
| `great`, `really liked it` | 4.5 | positive |
| `solid`, `pretty good` | 3.5 | positive/mixed |
| `fine`, `average` | 3.0 | mixed |
| `not great`, `disappointing` | 2.0 | negative |
| `terrible`, `hated it` | 1.0 | negative |

## Ambiguity rules

- If a phrase might refer to something other than the title, do not infer a rating.
- If the user says only `good` after a title lookup, ask whether they want to rate it only if a stored rating is needed.
- If the user rates an episode and the current conversation does not identify the show/season, ask for the show or season before writing.
- Always store `rating.raw` even when a normalized rating is inferred.
