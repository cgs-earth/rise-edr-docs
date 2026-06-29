---
title: "RISE API workflow"
hero_title: "API workflow guide"
lede: "Call the RISE EDR endpoint as raw HTTP, Python, raw R, or edr4r, with the same workflow shown four ways."
permalink: /raw-http/
---

This guide shows the same RISE EDR workflow in four toggleable forms:

- **HTTP**: URLs or request shapes you can paste into a browser, curl,
  Postman, or another HTTP client.
- **Python**: raw `requests` examples.
- **R (raw)**: raw `httr2` and `jsonlite` examples.
- **edr4r**: higher-level helpers that wrap the same EDR routes.

The examples use `https://data.usbr.gov/api/ogc` as the base URL. They
were checked against the public RISE EDR beta endpoint on June 27,
2026.

## 1. Set the base URL

Most API calls use `f=json`. Collection metadata, GeoJSON, and
CoverageJSON are all JSON-family responses.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 1 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

``` http
GET /api/ogc/?f=json HTTP/1.1
Host: data.usbr.gov
Accept: application/json
```

Full URL:

``` text
https://data.usbr.gov/api/ogc/?f=json
```

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
import requests

BASE = "https://data.usbr.gov/api/ogc"

response = requests.get(
    BASE + "/",
    params={"f": "json"},
    timeout=30,
)
response.raise_for_status()
landing = response.json()
```

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
library(httr2)
library(jsonlite)

base <- "https://data.usbr.gov/api/ogc"

response <- request(base) |>
  req_url_query(f = "json") |>
  req_perform()

landing <- resp_body_json(response, simplifyVector = FALSE)
```

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
library(edr4r)

rise <- edr_client("https://data.usbr.gov/api/ogc")
collections <- edr_collections(rise)
```

</div>
</div>

**Expected output:** A JSON landing page whose title is
`RISE EDR API - beta` and whose links include `collections`, `openapi`,
and `conformance`.

## 2. Inspect the RISE collection

The endpoint has one public EDR collection: `rise`. Inspect the
collection before requesting values so you know the supported routes and
parameter ids.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 2 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

``` sh
curl -sS 'https://data.usbr.gov/api/ogc/collections/rise?f=json'
```

Look for `data_queries` and `parameter_names`.

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
collection_response = requests.get(
    BASE + "/collections/rise",
    params={"f": "json"},
    timeout=30,
)
collection_response.raise_for_status()
collection = collection_response.json()

print(collection["title"])
print("queries:", list(collection.get("data_queries", {}).keys()))
print("parameter count:", len(collection.get("parameter_names", {})))
```

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
response <- request(base) |>
  req_url_path_append("collections", "rise") |>
  req_url_query(f = "json") |>
  req_perform()

collection <- resp_body_json(response, simplifyVector = FALSE)

collection$title
names(collection$data_queries)
length(collection$parameter_names)
```

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
collection <- edr_collection(rise, "rise")
names(collection$data_queries)

params <- edr_parameters(rise, "rise")
params[params$id %in% c("2", "3", "12", "18", "20", "1830"),
       c("id", "name", "unit_symbol")]
```

</div>
</div>

**Expected output:** The collection title is
`Reclamation Information Sharing Environment EDR`. The advertised EDR
data query is `locations`. The `parameter_names` object contains numeric
parameter ids.

## 3. Find RISE locations

Use `/items` when you want richer catalog metadata and filters. Use
`/locations` when you want the EDR location list for sampling.

This example finds Marys Lake by exact `locationName`.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 3 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

``` text
https://data.usbr.gov/api/ogc/collections/rise/items?f=json&locationName=Marys%20Lake&limit=5
```

Equivalent curl:

``` sh
curl -sS 'https://data.usbr.gov/api/ogc/collections/rise/items?f=json&locationName=Marys%20Lake&limit=5'
```

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
items_response = requests.get(
    BASE + "/collections/rise/items",
    params={
        "f": "json",
        "locationName": "Marys Lake",
        "limit": 5,
    },
    timeout=30,
)
items_response.raise_for_status()
items = items_response.json()

for feature in items["features"]:
    print(feature["id"], feature["properties"]["locationName"])
```

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
response <- request(base) |>
  req_url_path_append("collections", "rise", "items") |>
  req_url_query(
    f = "json",
    locationName = "Marys Lake",
    limit = 5
  ) |>
  req_perform()

items <- resp_body_json(response, simplifyVector = FALSE)

vapply(
  items$features,
  function(feature) paste(feature$id, feature$properties$locationName),
  character(1)
)
```

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
locations <- edr_locations(
  rise,
  "rise",
  bbox = c(-105.6, 40.3, -105.5, 40.4)
)

locations[, c("id", "name")]
```

</div>
</div>

**Expected output:** A GeoJSON `FeatureCollection`. Marys Lake has
feature id and `locationID` `1`, with coordinates
`[-105.5343083, 40.3440796]`.

## 4. Request one location's values

Once you know a location id, request a time series from
`/locations/{locId}`. This example requests Marys Lake (`1`) for
Lake/Reservoir Elevation (`2`) and Lake/Reservoir Storage (`3`) over a
short January 2024 window.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 4 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

``` http
GET /api/ogc/collections/rise/locations/1?f=json&datetime=2024-01-01/2024-01-07&parameter-name=2,3 HTTP/1.1
Host: data.usbr.gov
Accept: application/prs.coverage+json, application/json
```

Full URL:

``` text
https://data.usbr.gov/api/ogc/collections/rise/locations/1?f=json&datetime=2024-01-01/2024-01-07&parameter-name=2,3
```

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
series_response = requests.get(
    BASE + "/collections/rise/locations/1",
    params={
        "f": "json",
        "datetime": "2024-01-01/2024-01-07",
        "parameter-name": "2,3",
    },
    timeout=30,
)

if series_response.status_code == 204:
    coverage = None
else:
    series_response.raise_for_status()
    coverage = series_response.json()
```

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
response <- request(base) |>
  req_url_path_append("collections", "rise", "locations", "1") |>
  req_url_query(
    f = "json",
    datetime = "2024-01-01/2024-01-07",
    `parameter-name` = "2,3"
  ) |>
  req_perform()

coverage <- if (resp_status(response) == 204) {
  NULL
} else {
  resp_body_json(response, simplifyVector = FALSE)
}
```

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
marys_lake <- edr_location(
  rise,
  "rise",
  loc_id = "1",
  datetime = "2024-01-01/2024-01-07",
  parameter_name = "2,3"
)

marys_lake_tbl <- covjson_to_tibble(marys_lake)
```

</div>
</div>

**Expected output:** A CoverageJSON `Coverage` with `domainType`
`PointSeries`. The response contains time axis `t`, point axes `x` and
`y`, and two ranges keyed by parameter ids `2` and `3`.

## 5. Read the CoverageJSON

For Marys Lake, the live response shape is:

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
    "2": {"axisNames": ["t"], "values": [8034.23, 8034.24]},
    "3": {"axisNames": ["t"], "values": [698.739, 699.129]}
  }
}
```

The first elevation value belongs to the first timestamp. The first
storage value also belongs to the first timestamp.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 5 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

CoverageJSON aligns `domain.axes` and `ranges`:

``` text
domain.axes.t.values[0] pairs with ranges["3"].values[0]
domain.axes.t.values[1] pairs with ranges["3"].values[1]
```

Flatten the response before charting, joining, or exporting.

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
def point_series_rows(covjson):
    rows = []
    axes = covjson["domain"]["axes"]
    times = axes.get("t", {}).get("values", [])
    x_value = axes.get("x", {}).get("values", [None])[0]
    y_value = axes.get("y", {}).get("values", [None])[0]
    parameters = covjson.get("parameters", {})

    for parameter_id, range_obj in covjson.get("ranges", {}).items():
        name = parameters.get(parameter_id, {}).get("name")
        unit = (
            parameters.get(parameter_id, {})
            .get("unit", {})
            .get("symbol", {})
            .get("value")
        )

        for time_value, data_value in zip(times, range_obj.get("values", [])):
            rows.append({
                "location_id": covjson.get("id"),
                "parameter_id": parameter_id,
                "parameter_name": name,
                "unit": unit,
                "datetime": time_value,
                "x": x_value,
                "y": y_value,
                "value": data_value,
            })

    return rows

rows = point_series_rows(coverage)
```

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
point_series_rows <- function(covjson) {
  axes <- covjson$domain$axes
  times <- axes$t$values
  x_value <- axes$x$values[[1]]
  y_value <- axes$y$values[[1]]

  do.call(rbind, lapply(names(covjson$ranges), function(parameter_id) {
    values <- covjson$ranges[[parameter_id]]$values
    parameter <- covjson$parameters[[parameter_id]]
    unit <- parameter$unit$symbol$value

    data.frame(
      location_id = covjson$id,
      parameter_id = parameter_id,
      parameter_name = parameter$name,
      unit = unit,
      datetime = unlist(times),
      x = x_value,
      y = y_value,
      value = unlist(values)
    )
  }))
}

rows <- point_series_rows(coverage)
```

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
rows <- covjson_to_tibble(marys_lake)

rows[, c("coverage_id", "parameter", "datetime", "x", "y", "value")]
```

</div>
</div>

**Expected output:** A rectangular table with one row per location,
parameter, and timestamp.

## 6. Handle no-content responses

The collection-level parameter catalog is global. A parameter can be
valid for RISE but unavailable at a specific location or time. For
example, Marys Lake returned data for parameter `3` over the example
window, while a request for parameter `1830` over the same window
returned `204 No Content` when checked.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 6 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

``` sh
curl -i 'https://data.usbr.gov/api/ogc/collections/rise/locations/1?f=json&datetime=2024-01-01/2024-01-07&parameter-name=1830'
```

Check the HTTP status before assuming the body contains JSON.

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
response = requests.get(
    BASE + "/collections/rise/locations/1",
    params={
        "f": "json",
        "datetime": "2024-01-01/2024-01-07",
        "parameter-name": "1830",
    },
    timeout=30,
)

if response.status_code == 204:
    rows = []
else:
    response.raise_for_status()
    rows = point_series_rows(response.json())
```

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
response <- request(base) |>
  req_url_path_append("collections", "rise", "locations", "1") |>
  req_url_query(
    f = "json",
    datetime = "2024-01-01/2024-01-07",
    `parameter-name` = "1830"
  ) |>
  req_perform()

rows <- if (resp_status(response) == 204) {
  data.frame()
} else {
  point_series_rows(resp_body_json(response, simplifyVector = FALSE))
}
```

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
# If a valid RISE request has no values, the server may return HTTP 204.
# Wrap exploratory calls so no-content responses become empty tables in
# your analysis workflow.
```

</div>
</div>

**Expected output:** Either a CoverageJSON response with values or an
HTTP `204 No Content` response. Treat `204` as "no matching values for
this location, parameter, and time window", not as a broken collection.

## 7. Get all selected data in an area of interest

RISE does not currently advertise `area` or `cube`, so there is no
single "give me every value in this polygon or bbox" route. The
workaround is to build the AOI extract on the client:

1. Discover RISE locations in the AOI with `/items?bbox=...`.
2. Page through all matching items with `limit` and `offset`.
3. Loop over returned `locationID` values.
4. Request `/locations/{locId}` for the selected `datetime` and
   `parameter-name` values.
5. Flatten CoverageJSON and join it back to the item metadata.

The examples below use a small bbox around Marys Lake and Estes Power
Plant. Expand the bbox only after the workflow works at this scale.

<div class="example-tabs" data-code-tabs>
<div class="tab-buttons" role="tablist" aria-label="Step 7 examples">
<button type="button" class="tab-button" data-code-tab="http">HTTP</button>
<button type="button" class="tab-button" data-code-tab="python">Python</button>
<button type="button" class="tab-button" data-code-tab="r-raw">R (raw)</button>
<button type="button" class="tab-button" data-code-tab="edr4r">edr4r</button>
</div>

<div class="tab-panel" data-code-panel="http" markdown="1">

Discover locations in the AOI:

``` sh
curl -sS 'https://data.usbr.gov/api/ogc/collections/rise/items?f=json&bbox=-105.6,40.3,-105.5,40.4&limit=500&offset=0'
```

If `numberMatched` is greater than `numberReturned`, request the next
page by increasing `offset`:

``` text
offset=500
offset=1000
offset=1500
```

Then request values for each returned `locationID`:

``` sh
curl -sS 'https://data.usbr.gov/api/ogc/collections/rise/locations/1?f=json&datetime=2024-01-01/2024-01-07&parameter-name=2,3'
curl -sS 'https://data.usbr.gov/api/ogc/collections/rise/locations/3262?f=json&datetime=2024-01-01/2024-01-07&parameter-name=2,3'
```

</div>

<div class="tab-panel" data-code-panel="python" markdown="1" hidden>

``` python
from time import sleep

AOI_BBOX = "-105.6,40.3,-105.5,40.4"
TIME_WINDOW = "2024-01-01/2024-01-07"
PARAMETER_IDS = ["2", "3"]


def get_json(path, params, timeout=60):
    response = requests.get(BASE + path, params=params, timeout=timeout)

    if response.status_code == 204:
        return None

    response.raise_for_status()
    return response.json()


def iter_aoi_items(bbox, page_size=500):
    offset = 0

    while True:
        payload = get_json(
            "/collections/rise/items",
            {
                "f": "json",
                "bbox": bbox,
                "limit": page_size,
                "offset": offset,
                "properties": "locationID,locationName,timezoneID,elevation",
            },
        )
        features = payload.get("features", [])

        if not features:
            break

        yield from features

        offset += len(features)
        number_matched = payload.get("numberMatched")

        if number_matched is not None and offset >= number_matched:
            break
        if len(features) < page_size:
            break


aoi_rows = []

for feature in iter_aoi_items(AOI_BBOX):
    props = feature["properties"]
    loc_id = str(props["locationID"])

    coverage = get_json(
        f"/collections/rise/locations/{loc_id}",
        {
            "f": "json",
            "datetime": TIME_WINDOW,
            "parameter-name": ",".join(PARAMETER_IDS),
        },
    )

    if coverage is None:
        continue

    for row in point_series_rows(coverage):
        row.update({
            "locationName": props.get("locationName"),
            "timezoneID": props.get("timezoneID"),
            "elevation": props.get("elevation"),
        })
        aoi_rows.append(row)

    sleep(0.2)
```

The `point_series_rows()` helper is defined in the CoverageJSON section
above. Keep `PARAMETER_IDS` deliberate; do not start by requesting every
advertised parameter for every location.

</div>

<div class="tab-panel" data-code-panel="r-raw" markdown="1" hidden>

``` r
aoi_bbox <- "-105.6,40.3,-105.5,40.4"
time_window <- "2024-01-01/2024-01-07"
parameter_ids <- "2,3"
page_size <- 500

features <- list()
offset <- 0

repeat {
  response <- request(paste0(base, "/collections/rise/items")) |>
    req_url_query(
      f = "json",
      bbox = aoi_bbox,
      limit = page_size,
      offset = offset,
      properties = "locationID,locationName,timezoneID,elevation"
    ) |>
    req_perform()

  payload <- resp_body_json(response, simplifyVector = FALSE)
  batch <- payload$features

  if (length(batch) == 0) {
    break
  }

  features <- c(features, batch)
  offset <- offset + length(batch)
  number_matched <- payload$numberMatched

  if (!is.null(number_matched) && offset >= number_matched) {
    break
  }

  if (length(batch) < page_size) {
    break
  }
}

aoi_pieces <- list()

for (feature in features) {
  props <- feature$properties
  loc_id <- as.character(props$locationID)

  response <- request(paste0(base, "/collections/rise/locations/", loc_id)) |>
    req_url_query(
      f = "json",
      datetime = time_window,
      `parameter-name` = parameter_ids
    ) |>
    req_perform()

  if (resp_status(response) == 204) {
    next
  }

  coverage <- resp_body_json(response, simplifyVector = FALSE)
  rows <- point_series_rows(coverage)
  rows$locationName <- props$locationName
  rows$timezoneID <- props$timezoneID
  rows$elevation <- props$elevation

  aoi_pieces[[length(aoi_pieces) + 1]] <- rows
  Sys.sleep(0.2)
}

aoi_rows <- if (length(aoi_pieces)) {
  do.call(rbind, aoi_pieces)
} else {
  data.frame()
}
```

The `point_series_rows()` helper is defined in the CoverageJSON section
above. For larger AOIs, write each location's result to disk as it
finishes so a failed run can resume.

</div>

<div class="tab-panel" data-code-panel="edr4r" markdown="1" hidden>

``` r
aoi_locations <- edr_locations(
  rise,
  "rise",
  bbox = c(-105.6, 40.3, -105.5, 40.4)
)

parameter_ids <- "2,3"
time_window <- "2024-01-01/2024-01-07"

pieces <- lapply(aoi_locations$id, function(loc_id) {
  result <- try(
    edr_location(
      rise,
      "rise",
      loc_id = as.character(loc_id),
      datetime = time_window,
      parameter_name = parameter_ids
    ),
    silent = TRUE
  )

  if (inherits(result, "try-error") || is.null(result)) {
    return(NULL)
  }

  rows <- covjson_to_tibble(result)
  rows$location_id <- as.character(loc_id)
  rows
})

pieces <- Filter(Negate(is.null), pieces)

aoi_rows <- if (length(pieces)) {
  do.call(rbind, pieces)
} else {
  data.frame()
}
```

Use `/items` with raw HTTP/R/Python when you need the richer item
metadata (`timezoneID`, `elevation`, exact `locationName`) joined onto
the final table.

</div>
</div>

**Expected output:** A combined table of values for selected parameters
and times at all RISE locations discovered inside the AOI.

## 8. Scale up carefully

The RISE endpoint currently retrieves values by location. To work across
many sites, first discover locations, then loop over the specific ids
you need.

Good defaults while exploring:

- Use `f=json`.
- Keep bbox order as `min_lon,min_lat,max_lon,max_lat`.
- Start with one location, one or two parameters, and a short
  `datetime` interval.
- Check for `204 No Content`.
- Cache trusted responses before doing downstream analysis.
- Widen only one dimension at a time: locations, time, or parameters.

Use this route pattern for each selected location:

``` text
/collections/rise/locations/{locId}?f=json&datetime={start}/{end}&parameter-name={ids}
```

The RISE endpoint does not currently advertise `cube` or `area`, so do
not assume bbox-based value retrieval is available just because bbox
works for location discovery.
