# Markdown artifacts

The common case: a Markdown document becomes a rendered page at a stable URL.

## Publish

`publish_foleo_artifact` with an inline document:

```json
{
  "source": { "sourceFormat": "markdown", "content": "# Release notes\n\n…" },
  "idempotencyKey": "<uuid>"
}
```

The server derives the title and the slug from the document, so the returned
`url` is `https://<account-handle>.<foleo tenant domain>/<slug>`. Appending
`.md` to that URL serves the Markdown source. Both URLs survive updates.

New pages are **unlisted**: anyone with the link can read them, and nothing
advertises them. That is the only access state Markdown has today.

`name` is rejected for Markdown (`name_not_supported_for_markdown`). Names are
an active-HTML concept.

## Update

Pass the existing artifact and its current `etag`; the URL does not move.

```json
{
  "artifactId": "<id>",
  "etag": "<current etag>",
  "source": { "sourceFormat": "markdown", "content": "…" },
  "idempotencyKey": "<uuid>"
}
```

Read the artifact first if you are not certain the `etag` is current. Ask the
human before overwriting a published document they did not just hand you.

## Already published?

If creating returns `matching_source_exists`, Foleo already holds this content
for this account. Report the named candidates and update the one the user
picks. Do not create a second copy silently — a duplicate is a second URL to
keep in sync, and the user may already have shared the first.

## Settings that exist

`update_foleo_artifact_settings` takes a `patch` of settings the server lists
as mutable. For a Markdown document those are the presentation ones, including:

- `frontmatterDisplay`: `collapsed` (default), `expanded`, or `hidden` —
  how YAML front matter appears on the rendered page.
- `titleUserSet` / `slugUserSet`: pin the title or slug so later updates stop
  re-deriving them from the content.

`visibility` and `password` are **reserved** for Markdown and return
`not_implemented`. That is deliberate, not a gap to route around: Markdown has
no private mode, no password, and no public/indexed mode. If the user needs
access control, say so plainly.

## Things worth telling the user

- Mermaid fences render as diagrams in the reader's browser inside a sandboxed
  frame. They need JavaScript, they print only once loaded, and reader modes
  usually drop them — so a diagram must never carry information the surrounding
  text omits.
- Documents have a size ceiling; `get_foleo_capabilities` reports the current
  one under `contentSizeLimits.markdown`.
- Unpublishing makes reader URLs 404 while the owner keeps the history. It is
  the reversible way to take a link down, and is usually what "delete this"
  actually means. Confirm which one the user wants.
