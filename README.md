# SPA crawler

[![CI](https://github.com/hu553in/spa-crawler/actions/workflows/ci.yml/badge.svg)](https://github.com/hu553in/spa-crawler/actions/workflows/ci.yml)

- [License](./LICENSE)
- [How to contribute](./CONTRIBUTING.md)
- [Code of conduct](./CODE_OF_CONDUCT.md)

A CLI-friendly crawler that can **optionally log in**, **crawl a website**, and **mirror pages and static assets**
into a local directory so the result can be served by a static web server (Caddy, etc.).

The project targets modern SPAs and Next.js-style applications where content is rendered dynamically and
traditional tools like `wget` or `curl` often fail to capture working pages.

---

## Features

- Optional authentication flow
  - Fills login/password inputs
  - Submits the form
  - Waits for redirect after successful login

- Playwright-based rendering
  - Supports SPAs, hydration, and client-side routing
  - Handles dynamic content loading

- Mirrors HTML pages
  - Saved to `out/pages/**/index.html`

- Mirrors many static assets
  - Examples: `/_next/**`, `*.css`, `*.js`, images, fonts, etc.
  - Saved to `out/assets/**` and `out/assets_q/**`

- Single browser session / session pool
  - Designed to improve reliability for authenticated crawling

- Additional URL discovery
  - Extracts candidate links from rendered DOM
  - Reads Next.js `__NEXT_DATA__` from the page
  - Reads `/_next/data/**.json` payloads from intercepted responses
  - Helps discover routes referenced in JSON/JS, not only in `<a>` tags

- Redirect behavior capture (hybrid)
  - Collects HTTP redirect edges from observed 3xx chains
  - Collects client-side redirects when loaded URL changes in browser
  - Exports high-confidence Caddy redirect rules to `out/redirects.caddy`
  - Creates HTML redirect pages for missing source pages as a static-hosting fallback

---

## Output structure

```
out/
  redirects.caddy
  pages/
    index.html
    nested_page/index.html
    ...
  pages_q/
    search/
      page=2/index.html
    ...
  assets/
    _next/static/...
    logo.svg
    favicon.ico
    ...
  assets_q/
    _next/static/chunk.js/
      v=123
    ...
```

Typical serving layout:

- `out/pages` → HTML root
- `out/pages_q` → query HTML variants (e.g. `/search?page=2`)
- `out/assets` → static files root (or mounted under `/`, depending on server configuration)
- `out/assets_q` → query static variants (e.g. `/app.js?v=123`)
- `out/redirects.caddy` → generated Caddy `redir` rules from observed redirects
- `out/pages` and `out/pages_q` may include generated HTML redirect pages for missing sources

---

## Install

1. Install [uv](https://docs.astral.sh/uv/)
2. Install dependencies:
   ```
   make install_deps
   ```

---

## Usage

The crawler is implemented as:

- Async Python function `crawl(config)`
- Typer CLI wrapper

Basic flow:

```
make help
```

Then review these files for practical usage examples and deployment templates:

- `Makefile`
- `Dockerfile`
- `docker-compose.yml`
- `Caddyfile`

---

## CLI filtering defaults

- Include links: `{base_url}/**` when no include filters are provided
- Exclude links: login regex only (`.*{login_path}.*`) when `--login-required` is `true`
- API path prefixes: empty by default; add `--api-path-prefix` values if you want API routes excluded
  from page discovery, asset mirroring, and redirect collection

---

## Deployment of mirrored site

This project only produces a mirrored static copy of a website.
You must decide how and where to deploy or serve it.

Example deployment stack included:

- `Dockerfile`
- `docker-compose.yml`
- `Caddyfile`
- Environment configuration via `.env`

`Caddyfile` imports `/srv/redirects.caddy`.
`Dockerfile` creates a no-op placeholder for this file when it is absent.
`Caddyfile` also normalizes non-`GET`/`HEAD` methods by redirecting them to `GET` with `303` on the same URI
(helps avoid `405 Method Not Allowed` on static mirrors).

To use HTTP basic authentication with Caddy, generate a password hash:

```
caddy hash-password
```

Then set environment variables used by `Caddyfile`:

- `ENABLE_BASIC_AUTH=true`
- `BASIC_AUTH_USER=<username>`
- `BASIC_AUTH_PASSWORD_HASH=<output from previous command>`

### If you want a server other than Caddy

The repository ships only a Caddy serving configuration.
For any other server, you must re-implement the same URL-to-filesystem lookup behavior.

What must be ported from the `Caddyfile` logic:

- Page lookup without query: `/pages{path}` → `/pages{path}/index.html` → `/pages{path}.html`
- Page lookup with query: `/pages_q{path}/{query}` → `/pages_q{path}/{query}/index.html` → `/pages_q{path}/{query}.html`
  (with fallback to non-query pages)
- Asset lookup without query: `/assets{path}` → `/assets{path}.*` → `/assets{path}.bin`
- Asset lookup with query: `/assets_q{path}/{query}` (with fallback to non-query assets)
- Header policy: immutable cache for `/_next/*`, no-cache for mirrored HTML pages
- Method policy: non-`GET`/`HEAD` requests are redirected with `303` to the same URI before static lookup

Redirect support must also be ported:

- Current export is Caddy-specific (`out/redirects.caddy` with `redir` directives)
- For another server, add a converter step (from observed redirects to that server syntax)
  or implement a new Python exporter
- HTML redirect pages in `out/pages` and `out/pages_q` are server-agnostic fallbacks and should still work
  if lookup is ported correctly

For Nginx specifically, reproducing query-based lookup (`{query}` in the filesystem path) and fallback chains
usually requires `njs` or careful `map` + `try_files` composition.

---

## Limitations

This is a hobby / experimental project.
It aims to handle modern SPAs reasonably well but is **not a fully robust site mirroring solution**.

### Session configuration

Session behavior is currently hardcoded.
There are no CLI arguments to tune session pool settings or advanced browser session parameters.

Authenticated crawling may require manual code adjustments.

---

### High parallelism and memory usage

At high concurrency levels the crawler may:

- Consume large amounts of RAM
- Trigger repeated warnings about memory limits
- Become unstable or slower

Recommended approach:

- Use low concurrency
- For authenticated crawling, use concurrency = 1

### Hardware tuning

You can tune Crawlee memory behavior via environment variables:

- `CRAWLEE_MEMORY_MBYTES`: absolute memory limit (in MB) used by Crawlee autoscaling
- `CRAWLEE_MAX_USED_MEMORY_RATIO`: fraction of that limit that can be used before throttling

Example `.env` values:

```
CRAWLEE_MEMORY_MBYTES=20000
CRAWLEE_MAX_USED_MEMORY_RATIO=0.95
```

Tuning guidance:

- Lower values can reduce OOM risk on smaller machines
- Higher values can improve throughput on larger machines, but may increase RAM pressure

---

### Large number of HTTP errors in output

During crawling you may see large amounts of:

- 404 responses
- Failed asset requests
- Transient navigation errors

This is expected behavior for modern SPAs and does not necessarily indicate crawler failure.

The crawler intentionally prioritizes successful page mirroring rather than eliminating every failed request.

---

### Not all assets can be mirrored

The crawler downloads many static assets but **cannot guarantee full asset capture**.

Some resources may be skipped due to:

- Streaming or opaque responses
- Dynamically generated URLs
- Authentication-protected resources
- Browser caching behavior
- Implementation complexity
- Unsafe or ambiguous query strings for static-server mapping

The mirrored site may occasionally require manual fixes.

---

### URL discovery is heuristic

The crawler attempts to discover routes using:

- DOM extraction
- `__NEXT_DATA__` parsing
- `/_next/data/**.json` parsing

However, if a route is only accessible via complex client logic or hidden interactions,
it may never be discovered automatically.

Manual entrypoints may be required.

---

### Redirect export is observational

`out/redirects.caddy` and generated HTML redirect pages are based only on redirects observed during crawl.

This means:

- Paths never visited during crawl will not have redirect rules
- Ambiguous source URLs may be ignored if confidence is below threshold
- Export keeps only one best target per source URL

---

### Stability vs. completeness tradeoff

The project intentionally favors:

- Simplicity
- Maintainability
- Ease of experimentation

over:

- Perfect site replication
- Exhaustive browser instrumentation

---

## Tips / troubleshooting

### SPA login inputs reset while typing

Some SPAs rerender login forms during hydration.

Increase rerender timeout to allow DOM stabilization.

---

### Pages exist but never get crawled

Common causes:

- Routes exposed only via buttons or JS logic
- Routes hidden in JSON menus
- Conditional client routing

Possible fixes:

- Add include globs/regexes
- Add manual entrypoints via `--additional-crawl-entrypoint-url`
- Extend URL extraction logic for project-specific patterns

---

### Assets missing / CSS not loading

Assets are mirrored using Playwright request interception.

Some resource types cannot be reliably captured and will be skipped.

---

### Unexpected logout or broken authentication

Recommended configuration:

- Concurrency = 1
- Single session pool
- No session rotation

---

## Development status

This project is:

- Experimental
- Evolving
- Intentionally pragmatic rather than complete

It is useful for:

- Offline mirrors
- Testing mirrored SPAs
- Migration experiments
- Static hosting tests

It is **not** intended as a universal or production-grade website archiving solution.

---

## Ethics and legality

Only crawl content you are authorized to access and store.

Respect:

- Website terms of service
- Privacy rules
- Copyright and licensing restrictions

Do not use this tool to extract or redistribute restricted data without permission.
