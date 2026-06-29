---
title: "RISE endpoint guide"
hero_title: "RISE endpoint guide"
lede: "Understand the public USBR RISE EDR endpoint, its single collection, supported routes, and how to choose the right request."
permalink: /field-guide/
---

The [RISE EDR API](https://data.usbr.gov/api/ogc/) provides an
Open Geospatial Consortium Environmental Data Retrieval interface for
Bureau of Reclamation data available in RISE. From the RISE team
perspective, the endpoint is a standards-based front door for
time-series discovery and retrieval.

RISE is an operational data system and the API is labeled beta, so
re-run discovery before production analysis.

## 1. What the endpoint exposes

The endpoint currently advertises one collection, `rise`.

| Object | What to use | Notes |
|---|---|---|
| Landing page | `/` | Service links, OpenAPI, conformance, collections |
| Collections list | `/collections` | Confirms that the public EDR service has collection `rise` |
| RISE collection | `/collections/rise` | Collection metadata, parameter catalog, links, advertised query types |
| Location catalog | `/collections/rise/items` | GeoJSON features with `locationID`, `locationName`, `timezoneID`, `elevation`, and geometry |
| One catalog item | `/collections/rise/items/{featureId}` | Full metadata for one RISE location feature |
| EDR locations | `/collections/rise/locations` | Pre-defined sampling locations, usually simpler than `/items` |
| EDR values | `/collections/rise/locations/{locId}` | CoverageJSON time series for one location and one or more parameters |
| Queryables | `/collections/rise/queryables` | Filterable item properties |
| Schema | `/collections/rise/schema` | Feature property schema |
| OpenAPI | `/openapi` | Machine-readable endpoint reference |

The collection advertises `locations` as its EDR data query. It does
not currently advertise `cube`, `area`, `position`, or `radius` routes.
For multi-location work, discover candidate locations with `/items` or
`/locations`, then request each needed location id.

For an area of interest, use `/items?bbox=...` as the spatial
discovery step, page through every matching location, then loop over
those `locationID` values with `/locations/{locId}`. This produces the
same practical result as an AOI extract, but the aggregation happens in
your client code rather than in a single RISE EDR route.

## 2. Items versus locations

RISE exposes location-like information in two useful views.

`/items` is the richer OGC API Features view. Use it when you need
catalog metadata, pagination, CSV export, or filters such as
`locationName`.

Example item:

``` json
{
  "type": "Feature",
  "id": 1,
  "properties": {
    "locationID": 1,
    "locationName": "Marys Lake",
    "timezoneID": "MT",
    "elevation": 8050.0
  },
  "geometry": {
    "type": "Point",
    "coordinates": [-105.5343083, 40.3440796]
  }
}
```

`/locations` is the EDR sampling-location view. Use it when the next
step is to call `/locations/{locId}` for values.

Example EDR location:

``` json
{
  "type": "Feature",
  "id": 1,
  "properties": {
    "name": "Marys Lake"
  },
  "geometry": {
    "type": "Point",
    "coordinates": [-105.5343083, 40.3440796]
  }
}
```

The feature `id` and `locationID` are the ids used in the value route.
For example, Marys Lake is location `1`, so its value route begins:

``` text
https://data.usbr.gov/api/ogc/collections/rise/locations/1
```

## 3. Queryable fields

The RISE collection advertises these queryable item fields:

| Field | Type | How to use it |
|---|---|---|
| `geometry` | geometry | Spatial filtering and returned feature geometry |
| `locationID` | integer | Exact RISE location id |
| `locationName` | string | Exact location name filter, such as `Marys Lake` |
| `timezoneID` | string | Location time zone label, such as `MT` |
| `elevation` | number | Elevation metadata for the location |

The feature route also supports common OGC API Features controls such
as `bbox`, `limit`, `offset`, `properties`, and output format `f`.
While exploring, use low `limit` values and add filters one at a time.

## 4. Parameter catalog

The collection metadata includes a large `parameter_names` object. Each
key is the parameter id to send as `parameter-name`. The label is for
humans; the id is for requests.

Common operational examples include:

| Parameter id | Name | Unit | Typical use |
|---|---|---|---|
| `2` | Lake/Reservoir Elevation | `ft` | Reservoir water-surface elevation |
| `3` | Lake/Reservoir Storage | `af` | Reservoir storage volume |
| `12` | Lake/Reservoir Inflow | `cfs` | Average daily inflow |
| `18` | Lake/Reservoir Release - Total | `cfs` | Average daily total release |
| `20` | Streamflow | `cfs` | Average daily flow |
| `24` | Water Temperature | `DegF` | Average daily water temperature |
| `32` | Powerplant Generation | `MWh` | Daily hydropower generation |
| `33` | Precipitation | `in` | Total daily precipitation |
| `1830` | Lake/Reservoir Release - Total | `cfs` | Instant daily total release |

Other catalog entries cover water quality constituents, meteorological
measurements, snow, biological counts, hydropower, and related RISE
program data.

Important: the collection-level parameter catalog is global. It does
not mean every location has every parameter. If a location, parameter,
and time window have no matching values, the endpoint may return
`204 No Content`.

## 5. Discovery checklist

Start every session with discovery:

1. Open the landing page: `https://data.usbr.gov/api/ogc/?f=json`.
2. Confirm the collection: `/collections?f=json`.
3. Inspect RISE metadata: `/collections/rise?f=json`.
4. Find a location with `/items` or `/locations`.
5. Request values with `/locations/{locId}` using a short `datetime`
   range and one or two parameter ids.
6. Read the CoverageJSON axes and ranges before widening the query.

For the browser-friendly version of each route, use `f=html`. For
programmatic access, use `f=json`.

For AOI extracts, insert one more discovery step after item search:
paginate with `limit` and `offset` until all matching features are
collected, then use those feature ids as the value-retrieval queue.

## 6. Practical troubleshooting

| Symptom | Likely cause | What to try |
|---|---|---|
| `204 No Content` | The location has no values for that parameter and time range | Try a known parameter for the site, shorten the range, or inspect source RISE metadata |
| HTTP 404 on `cube` or `area` | The RISE endpoint does not advertise those routes | Use `/items` or `/locations` to discover locations, then query `/locations/{locId}` |
| Empty feature list | No item matched the bbox or property filter | Remove filters, lower `limit`, then add filters back one at a time |
| Unexpected parameter values | Parameter ids are numeric and sometimes similar | Re-check `parameter_names` in `/collections/rise?f=json` |
| Hard-to-read time series | EDR values are CoverageJSON arrays | Pair `domain.axes.t` with each requested range's `values` array |
| Browser URL fails but Python or R works | Reserved characters were not encoded | Let the client encode query parameters or encode spaces as `%20` |

Once a small request works, widen only one dimension at a time:
locations, time range, or parameter count. That keeps behavior
diagnosable and helps protect shared services.
