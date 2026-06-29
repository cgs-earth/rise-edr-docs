# USBR RISE EDR Guide

Standalone GitHub Pages source for RISE-focused OGC API - Environmental Data
Retrieval documentation.

## Edit content

- `index.md` is the landing page.
- `field-guide.md` explains the public RISE EDR endpoint and its single
  `rise` collection.
- `concepts.md` maps OGC EDR concepts to RISE locations, parameters, items,
  and CoverageJSON responses.
- `raw-http.md` walks through the RISE workflow as HTTP, Python, raw R, and
  `edr4r` examples.
- `_layouts/default.html` controls the page frame.
- `assets/css/site.css` controls the USBR-themed styling.

All tutorial pages are Markdown-editable and build as static HTML. The build
does not call the live RISE API.

## Publish on GitHub Pages

1. Create a new GitHub repository.
2. Push this directory as the repository root.
3. In repository settings, enable GitHub Pages with **GitHub Actions** as the
   source or use GitHub's built-in Pages/Jekyll publishing.
4. Push to `main`.

## Local preview

With Ruby 3.x available:

```sh
gem install jekyll
jekyll serve
```

Then open <http://127.0.0.1:4000/>.

## Source links

The content is tailored to the public USBR RISE EDR endpoint:

- <https://data.usbr.gov/api/ogc/>
- <https://data.usbr.gov/api/ogc/collections/rise>
- <https://data.usbr.gov/api/ogc/openapi>
