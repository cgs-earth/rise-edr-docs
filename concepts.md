---
title: "EDR concepts for RISE"
hero_title: "EDR concepts"
lede: "A practical map from OGC API - Environmental Data Retrieval terms to RISE locations, parameters, feature metadata, and time-series values."
permalink: /concepts/
---

OGC API - Environmental Data Retrieval, usually shortened to EDR, is a
standard way to ask environmental datasets for values by place, time,
variable, and query type. In the RISE endpoint, the main query type is
`locations`: find a named RISE location, then request values for that
location.

This page explains the concepts behind the endpoint at
`https://data.usbr.gov/api/ogc/`.

## 1. Discovery versus retrieval

EDR is easiest to understand if you separate two jobs:

| Job | Question | RISE routes |
|---|---|---|
| Discovery | What service, collection, locations, fields, and parameters exist? | `/`, `/collections`, `/collections/rise`, `/queryables`, `/schema`, `/items`, `/locations` |
| Retrieval | What values are available for this location, time window, and parameter? | `/collections/rise/locations/{locId}` |

Discovery responses are JSON metadata or GeoJSON features. Retrieval
responses are CoverageJSON, which stores values in arrays aligned to
coordinate axes.

## 2. The core objects

| EDR concept | RISE meaning | Example |
|---|---|---|
| Service | The whole public RISE EDR beta API | `https://data.usbr.gov/api/ogc` |
| Collection | The dataset exposed through the service | `rise` |
| Item | A GeoJSON catalog feature for a RISE location | `/collections/rise/items/1` |
| Location | A pre-defined EDR sampling point | `/collections/rise/locations/1` |
| Location id | The RISE location identifier used in URLs | `1` for Marys Lake |
| Parameter | A measured, calculated, or reported variable | `3` for Lake/Reservoir Storage |
| Coverage | The response object containing values | CoverageJSON `Coverage` |
| Axis | A coordinate dimension in the coverage domain | `t`, `x`, `y` |
| Range | The values for a requested parameter | `ranges["3"].values` |

The collection id is always `rise` for this endpoint. The location id
and parameter ids change based on what data you want.

## 3. Items and locations are related

RISE exposes both OGC API Features and OGC EDR views over the same
location inventory.

Use `/items` when you want richer location metadata:

``` text
/collections/rise/items?f=json&locationName=Marys%20Lake
```

The item response includes properties such as `locationID`,
`locationName`, `timezoneID`, and `elevation`.

Use `/locations` when you want the EDR sampling-location list:

``` text
/collections/rise/locations?f=json&bbox=-105.6,40.3,-105.5,40.4
```

The location response is intentionally simpler: id, display name, and
geometry. The id is what you send to `/locations/{locId}`.

## 4. Parameters are numeric ids

RISE parameter ids are numeric strings in the collection metadata.
Request them with the query parameter `parameter-name`.

``` text
parameter-name=3
```

Multiple parameters can be requested as a comma-separated list:

``` text
parameter-name=2,3
```

Always use the exact id from `/collections/rise?f=json`. Similar labels
can have different ids because they differ by statistic, unit, time
step, or source convention. For example, RISE advertises multiple
reservoir storage and elevation concepts across daily, monthly, and
sub-daily data.

## 5. Global catalog versus local availability

The collection advertises the global parameter catalog for RISE. That
catalog is broad because RISE contains many data programs. A specific
location usually has only a subset of those parameters.

This means:

- `parameter_names` tells you what parameter ids the collection knows.
- `/locations/{locId}` tells you whether a particular location has
  values for a parameter in a specific time window.
- `204 No Content` is a valid "no matching values" result for an
  otherwise well-formed request.

A good first request uses one known location, one common parameter, and
a short time window.

## 6. CoverageJSON response shape

A RISE value request such as:

``` text
https://data.usbr.gov/api/ogc/collections/rise/locations/1?f=json&datetime=2024-01-01/2024-01-07&parameter-name=3
```

returns a CoverageJSON `Coverage` shaped like this:

``` json
{
  "id": "1",
  "type": "Coverage",
  "domain": {
    "domainType": "PointSeries",
    "axes": {
      "t": {
        "values": [
          "2024-01-06T06:00:00",
          "2024-01-05T06:00:00"
        ]
      },
      "x": {"values": [-105.5343083]},
      "y": {"values": [40.3440796]}
    }
  },
  "ranges": {
    "3": {
      "axisNames": ["t"],
      "values": [698.739, 699.129]
    }
  }
}
```

Read it in three passes:

1. `domain.domainType` says this is a point time series.
2. `domain.axes` gives the coordinates and timestamps.
3. `ranges` gives values keyed by parameter id.

The first value in `ranges["3"].values` belongs to the first timestamp
in `domain.axes.t.values`. RISE may return timestamps in reverse
chronological order, so sort the flattened table if your analysis needs
ascending time.

## 7. Mapping routes to familiar tasks

| Familiar task | RISE route | Response |
|---|---|---|
| See API links and service metadata | `/?f=json` | JSON landing page |
| Confirm the collection | `/collections?f=json` | JSON collection list |
| Inspect RISE metadata and parameter ids | `/collections/rise?f=json` | JSON collection document |
| See filterable feature fields | `/collections/rise/queryables?f=json` | JSON Schema-like document |
| Find locations by name, bbox, or pagination | `/collections/rise/items?f=json` | GeoJSON FeatureCollection |
| Get one location's catalog metadata | `/collections/rise/items/{featureId}?f=json` | GeoJSON Feature |
| List EDR sampling locations | `/collections/rise/locations?f=json` | GeoJSON FeatureCollection |
| Retrieve one location's values | `/collections/rise/locations/{locId}?f=json&datetime=...&parameter-name=...` | CoverageJSON |

## 8. Working around no area or cube query

The RISE endpoint does not currently advertise EDR `area` or `cube`
queries. That means an area-of-interest request is a two-phase client
workflow rather than a single server-side coverage request.

The pattern is:

1. Use `/collections/rise/items?bbox=...` to find all RISE locations in
   the AOI.
2. Page through `/items` with `limit` and `offset` until all matching
   features are collected.
3. Keep each feature's `locationID`, `locationName`, geometry,
   `timezoneID`, and `elevation`.
4. Choose the parameter ids and time window you need.
5. Loop over the AOI location ids and call
   `/collections/rise/locations/{locId}`.
6. Treat `204 No Content` as "no values for this location, parameter,
   and time window".
7. Flatten each CoverageJSON response and join the rows back to the
   location metadata from `/items`.

In other words, bbox works for discovering locations, while
`/locations/{locId}` is the value-retrieval route.

``` text
AOI bbox
  -> /items?bbox=...&limit=...&offset=...
  -> location ids
  -> /locations/{locId}?datetime=...&parameter-name=...
  -> CoverageJSON per location
  -> flattened AOI table
```

Be precise about what "all data" means. The collection-level parameter
catalog is global and large. For most AOI workflows, "all data" should
mean all values for a selected parameter set and time interval across
the locations in the AOI. If you truly need every advertised parameter
for every location in a large area, batch the work by parameter family
and time window, cache intermediate results, and coordinate with the
RISE team if the extraction becomes operationally large.

## 9. The mental model in one pass

For a typical RISE time-series request:

1. Open the `rise` collection metadata.
2. Choose a parameter id from `parameter_names`.
3. Find a RISE location with `/items` or `/locations`.
4. Request `/locations/{locId}` with `datetime` and `parameter-name`.
5. Flatten CoverageJSON axes plus ranges into rows.
6. Join rows back to `/items` metadata when you need names, elevation,
   or time zone context.

For an AOI request:

1. Search `/items` with a bbox and page through all matches.
2. Use the returned `locationID` values as your work queue.
3. Retrieve values per location with `/locations/{locId}`.
4. Combine the per-location tables into one AOI table.

For code examples in HTTP, Python, raw R, and `edr4r`, continue to the
[API workflow guide]({{ '/raw-http/' | relative_url }}).
