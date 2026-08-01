# THE PAGE

A fixture page that exercises every markdown feature the renderer supports, so styling and cache behaviour can be checked against one document instead of hunting for a real page that happens to use the right elements.

Cache marker: **rev-001** — bump this, push, and the page should repaint with the new value once the webhook lands. If it still reads the old revision, the entry is being served stale.

## Getting started

The pipeline pulls every `.md` file under `docs/`, renders it, and stores the resulting HTML against a key derived from the repo and the path. A top-level file lands at `/docs/<repo>/<file>`; a file inside a section directory picks up the directory as a segment.

Install the toolchain, then point the watcher at a checkout:

```bash
cargo install cargo-leptos
cargo leptos watch --hot-reload
```

The first run warms the cache. Every run after that reads from it unless a webhook has invalidated the key, which is the behaviour this page exists to confirm.

### Configuration

Configuration is read once at boot. Missing keys fall back to the defaults in the table below, so a blank environment still starts.

| Variable | Default | Notes |
| --- | --- | --- |
| `DOCS_ROOT` | `docs/` | Directory scanned for sources |
| `CACHE_TTL` | `3600` | Seconds before an entry goes cold |
| `WEBHOOK_SECRET` | — | Required; startup fails without it |
| `MAX_BODY` | `1048576` | Rejects payloads above this size |

Alignment is worth checking too, since the renderer emits it as inline styles:

| Left | Center | Right |
| :--- | :----: | ----: |
| one | two | 3 |
| a much longer cell that pushes the column | mid | 42 |

### Authentication

Every request carries a signature header. The value is an HMAC of the raw body, computed with the shared secret, and it is compared in constant time — a plain `==` here would leak timing.

> **Note**
> Compare the digest, never the hex string. Decoding first and comparing bytes avoids a whole class of subtle mistakes around case and padding.
>
> The same applies to the replay window: reject anything older than five minutes rather than trusting the clock on the sending side.

## Working with sections

A section is a directory. The directory name becomes the URL segment, and `overview.md` inside it becomes the landing page for that segment. Anything else in the directory becomes a child page.

- Sections may not nest — one level only
- A section without an `overview.md` falls back to its first child
- Files beginning with `_` are skipped entirely
- Ordering follows the numeric prefix when one is present, then alphabetical

Nesting works as expected:

- Top level item
  - A child that inherits the parent's marker style
  - Another child, this one long enough to wrap onto a second line so the hanging indent is visible
- Back to the top level

Ordered lists carry their own rhythm:

1. Read the directory
2. Render each file
   1. Extract the heading tree
   2. Rewrite relative links
   3. Slug every heading for the table of contents
3. Write the results to the cache
4. Publish an invalidation event

### Checklist

Task lists render as real checkboxes, disabled and non-interactive:

- [x] Parse frontmatter
- [x] Render fenced code with line numbers
- [ ] Resolve cross-repo links
- [ ] Emit a sitemap
- [ ] ~~Ship a search index~~ — deferred to a later pass


# Shit stinky

Hello

## Code

Inline code such as `Section::items`, `--cache-ttl`, or a path like `src/api/repo_cache.rs` sits in the flow of a sentence without breaking the line rhythm.

A short block:

```rust
pub struct Section {
    pub title: Option<String>,
    pub source: Option<String>,
    pub items: Option<Vec<Section>>,
}
```

A block with a line long enough to scroll horizontally, which is the case worth watching — the gutter should stay pinned while the code slides underneath it:

```rust
fn cache_key(repo_id: &str, section: Option<&str>, file: Option<&str>) -> String {
    match (section, file) {
        (Some(s), Some(f)) => format!("{repo_id}:docs:{s}:{f}.md"),
        (Some(s), None) => format!("{repo_id}:docs:{s}:overview.md"),
        (None, Some(f)) => format!("{repo_id}:docs:{f}.md"),
        (None, None) => format!("{repo_id}:docs:overview.md"),
    }
}

// a deliberately long line so the block overflows its column and has to scroll sideways rather than wrap or push the page
let response = client.get(&url).header("authorization", format!("Bearer {token}")).timeout(Duration::from_secs(10)).send().await?;
```

And a payload, for a third language:

```json
{
  "repo": "personal-site",
  "event": "push",
  "paths": ["docs/example.md", "docs/specs/003-dyndocs/overview.md"],
  "invalidated": 2
}
```

## Links and text

Body copy carries **bold text** for emphasis, *italics* for terms being introduced, ***both together*** where something really needs it, and ~~strikethrough~~ for anything withdrawn. Inline links point [at another page](/docs), [at a heading on this one](#configuration), and [off-site entirely](https://commonmark.org/).

A bare URL is the case that used to force the column wider: https://example.com/a/deliberately/long/path/that/keeps/going/without/any/break/opportunity/at/all?query=1&more=2

Autolinks in angle brackets resolve too: <https://spec.commonmark.org/> and <docs@example.com>.

---

## Reference

Below the rule is the tail of the page — enough content that the table of contents has something to scroll against and the sticky panes can be watched while the reading column moves.

#### A fourth-level heading

Fourth-level headings appear in the prose but not the table of contents, which only tracks the first two levels.

![A placeholder image](https://placehold.co/1200x520/1d1c1a/908e89/png?text=Example+Figure)

Images are constrained to the measure and pick up the same border treatment as other blocks.

### Edge cases

A paragraph immediately followed by a code block, with no blank prose between them:

```
plain fence, no language — still gets a numbered gutter
```

A blockquote holding a list:

> Three things to verify on every deploy:
>
> 1. The cache key includes the section
> 2. Stale entries expire rather than linger
> 3. A failed render never overwrites a good entry

A final paragraph so the document does not end on a nested block, giving the last heading somewhere to scroll to when its anchor is clicked from the table of contents.

