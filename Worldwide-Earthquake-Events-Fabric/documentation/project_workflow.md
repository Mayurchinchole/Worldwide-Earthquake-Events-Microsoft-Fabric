# Project workflow

## Data source
USGS FDSN Event Web Service:
https://earthquake.usgs.gov/fdsnws/event/1/query

The Bronze notebook constructs a GeoJSON request using `start_date` and `end_date`.

## Layer details

| Layer | Input | Processing | Output |
|---|---|---|---|
| Bronze | USGS GeoJSON API | Extract `features`; persist JSON | Lakehouse `Files/<start_date>_earthquake_data.json` |
| Silver | Bronze JSON | Flatten selected nested fields; convert epoch milliseconds to timestamps | `earthquake_events_silver` |
| Gold | Silver table | Derive country code; classify significance | `earthquake_events_gold` |

## Gold significance rules

| Condition | `sig_class` |
|---|---|
| `sig < 100` | Low |
| `100 <= sig < 500` | Moderate |
| `sig >= 500` | High |

## Operational considerations

- Attach the intended Lakehouse before running notebooks.
- Supply `start_date` and `end_date` through notebook parameters or the calling pipeline.
- Install/enable `reverse_geocoder` in the Fabric environment before Gold processing.
- Current Silver and Gold writes use append mode. Add deduplication or merge logic before scheduling repeated overlapping loads.
