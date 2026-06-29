---
title: USBR RISE EDR Guide
hero_title: USBR RISE EDR Guide
lede: A standalone, Markdown-editable guide for exploring the RISE OGC API - Environmental Data Retrieval beta endpoint.
---

The RISE team publishes a standards-based OGC API - Environmental Data
Retrieval endpoint for Bureau of Reclamation time-series data at
`https://data.usbr.gov/api/ogc/`. The endpoint gives analysts,
application developers, and partner systems a consistent way to discover
RISE locations, inspect parameters, and request values as CoverageJSON.

This site is focused only on the public RISE EDR endpoint. It does not
cover the broader Western Water Datahub service or other provider
collections.

<div class="card-grid">
  <section class="card">
    <h2>Endpoint Guide</h2>
    <p>Start here for the service shape, supported RISE routes, location metadata, and parameter catalog guidance.</p>
    <p><a class="button-link" href="{{ '/field-guide/' | relative_url }}">Open endpoint guide</a></p>
  </section>
  <section class="card">
    <h2>EDR Concepts</h2>
    <p>Learn how RISE locations, items, parameters, axes, ranges, and CoverageJSON fit together.</p>
    <p><a class="button-link" href="{{ '/concepts/' | relative_url }}">Open concepts</a></p>
  </section>
  <section class="card">
    <h2>API Workflow</h2>
    <p>Call the same RISE workflow as HTTP, Python, raw R, or <code>edr4r</code> with toggleable examples.</p>
    <p><a class="button-link" href="{{ '/raw-http/' | relative_url }}">Open API workflow</a></p>
  </section>
</div>

## What this site is

- A standalone GitHub Pages-ready site for the RISE EDR beta endpoint.
- Plain Markdown source that can be edited directly in GitHub.
- Static documentation: no live API calls are made while building the site.
- Written from the USBR/RISE team perspective for users who want to
  discover and retrieve Reclamation time-series data through OGC EDR.

## Current endpoint summary

The public RISE EDR endpoint checked on June 27, 2026 advertises one
collection:

| Field | Value |
|---|---|
| Service title | `RISE EDR API - beta` |
| Base URL | `https://data.usbr.gov/api/ogc` |
| Collection id | `rise` |
| Collection title | `Reclamation Information Sharing Environment EDR` |
| Advertised EDR query | `locations` |
| Feature catalog routes | `/items`, `/items/{featureId}`, `/queryables`, `/schema` |

The collection-level parameter catalog advertised 782 parameter ids when
checked. Those ids cover reservoir operations, streamflow, meteorology,
snow, water quality, hydropower, and biological monitoring. A location
usually has only a subset of those parameters, so value requests should
start small.

## Key links

- [RISE EDR landing page](https://data.usbr.gov/api/ogc/)
- [RISE collection metadata](https://data.usbr.gov/api/ogc/collections/rise)
- [RISE OpenAPI reference](https://data.usbr.gov/api/ogc/openapi?f=html)
- [RISE queryables](https://data.usbr.gov/api/ogc/collections/rise/queryables?f=html)
- [RISE items as GeoJSON](https://data.usbr.gov/api/ogc/collections/rise/items?f=json&limit=5)
