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

## Build

The portable build contract is:

```sh
bundle install
script/build-site
```

The build writes static HTML to `public/`. Any git host or CI system can
publish that directory.

Set `SITE_BASEURL` when the site is hosted from a subpath:

```sh
SITE_BASEURL="/rise-edr-docs" script/build-site
```

For a custom domain hosted at the domain root, use an empty base URL:

```sh
SITE_BASEURL="" script/build-site
```

## Publish on GitHub Pages

This repository includes `.github/workflows/pages.yml`. On pushes to `main`,
GitHub Actions:

1. Installs Ruby dependencies from `Gemfile`.
2. Runs `SITE_BASEURL="/rise-edr-docs" script/build-site`.
3. Uploads the generated `public/` directory as the Pages artifact.
4. Deploys the artifact to GitHub Pages.

In repository settings, set **Pages** to use **GitHub Actions** as the source.

The expected GitHub Pages URL is:

<https://cgs-earth.github.io/rise-edr-docs/>

## Port to GitLab Pages

Use the same build command in GitLab CI and publish the `public/` artifact.
A minimal GitLab CI job would be:

```yaml
image: ruby:3.3

variables:
  SITE_BASEURL: "/rise-edr-docs"

pages:
  script:
    - bundle install
    - script/build-site
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

## Local preview

With Ruby 3.0 or newer available:

```sh
bundle install
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000/>.

## Source links

The content is tailored to the public USBR RISE EDR endpoint:

- <https://data.usbr.gov/api/ogc/>
- <https://data.usbr.gov/api/ogc/collections/rise>
- <https://data.usbr.gov/api/ogc/openapi>
