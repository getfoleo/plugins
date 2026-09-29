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

New pages are **unlisted** by default: anyone with the link can read them, and
nothing advertises them. Pass `settings: { "visibility": "private" }` to publish
a **private** document instead. It keeps the same `url`, which opens only for
signed-in members of the owning Foleo organization: everyone else sees a
not-found page with a Sign in button, and a member who signs in lands on the
document. Read the result's `access` before describing the link. A Markdown
publish never waits for approval under any connection policy.

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

For a small change, send `edits` instead of the whole document:

```json
{
  "artifactId": "<id>",
  "etag": "<current etag>",
  "source": {
    "sourceFormat": "markdown",
    "edits": [{ "find": "exact old text", "replace": "new text" }],
    "baseContentHash": "<sha256 of the current source>"
  },
  "idempotencyKey": "<uuid>"
}
```

The result's `contentHash` is the sha256 of the new document.

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

- `visibility`: `unlisted` or `private`. The URL never changes. Making a
  document private applies at once. Making a private document unlisted widens
  access, so it follows this connection's policy: it applies at once under
  `private_and_unlisted`, and otherwise returns `approval_required` for the
  human.

`password`, and the `password` and `public` visibilities, return
`not_implemented` for Markdown. That is deliberate, not a gap to route around
and not worth a retry: Markdown has no password and no public/indexed mode. If
the user needs either, say so plainly.

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
