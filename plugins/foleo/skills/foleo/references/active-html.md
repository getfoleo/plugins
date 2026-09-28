# Active HTML artifacts

Active HTML is executable: it runs JavaScript, shows forms, and can start
downloads. Foleo hosts it only for explicitly enrolled accounts. The human
sets each connection's publishing policy in Foleo; the agent cannot change it.

## Before anything else

Check `get_foleo_capabilities`. If `availability.namedAlphaHtml` is false, this
account cannot publish active HTML — say so and stop. Calling the tools anyway
returns `html_unavailable`. Enrollment is an operator decision, never something
to ask the server for twice.

The human sets this connection's publishing policy in Foleo Access;
`get_foleo_capabilities` returns it as `connectionPolicy`.

| Policy | Publishes in one call when clean |
| --- | --- |
| `private` (default for new connections) | Private pages |
| `private_and_unlisted` | Private and unlisted pages, open or contained |
| `private_and_contained_unlisted` | Private pages, and unlisted pages under the contained profile |
| `ask_each_time` | Nothing; every HTML page needs the human's approval |

Anyone with an unlisted URL can read it. Under every policy, a safety match,
a public page and a password page need the human.

Use top-level `runtimeProfile: "auto" | "open" | "contained"` on an HTML publish.
`auto` is the default: Foleo checks references and chooses contained only when
they fit. Explicit `open` stays open. Explicit `contained` fails with
`contained_profile_unavailable` and offending references if it cannot fit;
remove or self-host those references, or explicitly choose open and use Foleo
approval. The result reports the applied profile and `profileNote` separately
from staff/alpha eligibility. Contained blocks external fetch/XHR/WebSocket/
beacons, form submission, frames, plugins, remote images/media, workers and
non-allowlisted sources. It does not stop top-level navigation, `window.open`,
WebRTC, data in CDN request paths, or pre-PSL shared cookies. Never describe it
as non-exfiltrating.

## The approval loop

Publishing HTML over MCP stages exact bytes first. A page the connection's
policy covers, with a clear safety gate, can then publish in that same call. A
safety match needs
human review.

1. Call `publish_foleo_artifact` with an inline single-file page:

   ```json
   {
     "source": { "sourceFormat": "html", "content": "<!doctype html>…" },
     "name": "<slug>",
     "idempotencyKey": "<uuid>"
   }
   ```

   If the result says `published`, give the human its `url` and explain its
   `access`, `runtimeProfile`, `profileNote` and undo. If it says `approval_required`, nothing
   is live: the server staged the exact bytes and returned an `approvalId`.
   `approvalReason` says why: `safety_review` (the safety check flagged the
   page; see `safetyFindings`), `beyond_connection_policy` (the policy does not
   cover this visibility or profile) or `connection_asks_each_time`. `next`
   names the policy that would cover requests like this; tell the human, who
   alone can change it in Foleo Access.

2. For `approval_required`, give the human the `reviewUrl`. They see the exact source and its hash on
   a Foleo page — never on the artifact's own domain — and approve or reject.
   `get_foleo_publish_approval` tells you where it stands; poll it politely,
   and never pretend an approval happened.

3. Once approved, call `commit_foleo_publish` with the `approvalId` and a stable
   `idempotencyKey`. Only now does the artifact exist.

To update part of a published single-file page, send
`source: { "sourceFormat": "html", "edits": [...], "baseContentHash": "<byteHash>" }`
with `artifactId` and `etag`; a large page can arrive as
`source: { "sourceFormat": "html", "uploadId": "…", "sha256": "…" }` after
`stage_foleo_upload`. Foleo builds the exact resulting page and it goes
through this same loop: the same safety check, profile choice, policy and
approval, and `approvedContentHash` is the sha256 of the resulting page.

An approval is bound to this connection, organization, operation, content
revision, security settings and exact content hash. A title-only edit can
survive; content, access, name, status, policy or eligibility drift cannot.
An approval expires after 30 minutes. Within 24 hours of creation, an expired
candidate can be restaged from retained bytes with `restageFromApprovalId` and
an `idempotencyKey` alone. Never send content or changed settings alongside it.

Multi-page sites and local asset bundles are not an MCP operation. That is what
the Foleo CLI is for; say so instead of inlining a build.

## A Claude artifact on Foleo

When the human says "put this Claude artifact on Foleo":

1. **Get the authored HTML.** Use the file you wrote, or read the artifact's
   own source. Foleo's server cannot open a claude.ai link, private or not.
   A React or multi-part artifact must first be built into one self-contained
   HTML page; a Markdown artifact takes the Markdown path instead, without
   `origin`.
2. **Make it a complete document — you, not Foleo.** Add whatever is missing:
   `<!doctype html>`, `<meta charset="utf-8">`,
   `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`
   and a useful `<title>`. Inline relative assets. For mobile layouts, put
   safe-area padding on an inner container; never add `overflow`, `height` or
   `100dvh` rules to `html`/`body`, and no `position: sticky`/`fixed` chrome.
   Foleo stores and serves your exact bytes and never inserts tags or shims.
   If the human prefers the authored bytes unchanged, send them unchanged.
3. **Hash the final file** (sha256) before publishing.
4. **Publish privately**, in one call under a covered policy:

   ```json
   {
     "source": { "sourceFormat": "html", "content": "<!doctype html>…" },
     "settings": { "visibility": "private", "title": "<title>" },
     "origin": {
       "provider": "claude_artifact",
       "url": "https://claude.ai/artifact/<id>"
     },
     "idempotencyKey": "<uuid>"
   }
   ```

   `origin` is optional provenance: HTTPS, host exactly `claude.ai`, path
   `/artifact/<id>`, no query or fragment, 512 bytes at most. Foleo records it
   with the artifact, shows "From Claude" in the library, and never fetches
   it. An update may repeat the recorded origin or leave it out; a different
   one fails with `origin_conflict`.

5. **Check and hand over.** `approvedContentHash` must equal your sha256. Give
   the human `url` (it opens for the owning Foleo organization after sign-in) and
   `dashboardUrl`.

If `warnings` contains `claude_runtime_unavailable`, the page calls APIs only
a Claude artifact host provides (`window.claude`, artifact storage, database,
sampling, connected apps or files). They do not exist on Foleo, so those
parts will not work there. Say so, and replace them (for example with local
state) before promising the same behavior. The check is best-effort and
informational: it never blocks a publish or replaces human review.

Foleo's title setting changes the library and link preview only. To change the
page's own `<title>` or headings, edit the source and publish it again.

## Names are the address

An active-HTML artifact lives on its own subdomain. There are two kinds of
address:

- **Scoped (the default for a page you create):** `<slug>--<handle>`, for
  example `quarterly-report--yourhandle.foleo.site`. `name` is the slug. Scoped
  addresses are not capped by the name quota. If that address is taken for
  this handle, Foleo claims `slug-2` up to `slug-20` and returns the address it
  chose (`name`, `url`, `suffixed`).
- **Vanity (opt-in with `address: "vanity"`):** a short global name such as
  `quarterly-report.foleo.site`. `name` is the whole label, and each one
  uses a daily name claim and an active-name slot. Ask the human before
  spending one. Foleo never switches the address class for you.

This is a change: `name` without `address` used to claim a vanity name.

`name` may be omitted; Foleo derives the slug (or vanity label) from
`settings.title` or the page's `<title>` and returns it. Rules: 3–63
characters, `a-z`, `0-9`, and hyphens, with no leading, trailing, or doubled
hyphen. Either kind of address can be guessed from the title, so unlisted still
means anyone who knows or guesses the address can read it.

To check a name without sending the page, call `check_foleo_artifact_name`
with `address` and a `name` or `title` (or `publish_foleo_artifact` with
`preview: true` and no `source`). It returns the address a create would claim
(`previewUrl`), its status, a `suggestedSlug` and the vanity quota cost. It
reserves nothing, and the publish can still find the address taken.
`address` on an update or on Markdown returns `address_not_applicable`: an
update keeps its address.

Renaming later is an explicit decision about the old URL, and it is the
human's to make, not yours:

- `rename_foleo_artifact_name` with `previousLabel: "keep-as-alias"` keeps the
  old URL working, and the kept alias keeps its own kind. To rename to another
  scoped address, pass the whole `<slug>--<handle>` label as `name`.
- `previousLabel: "release"` stops the old URL working and holds the name for
  this account for a cooldown period.
- `release_foleo_artifact_name` drops an extra name and requires
  `acknowledgeLinkBreakage: true`. An artifact's last name cannot be released.

`list_foleo_artifact_names` shows what the account holds and its name quota.

## Settings

For active HTML, `visibility` and `password` are real: `unlisted` shares by
link, `private` conceals the page, and setting a password switches visibility
to `password`. Public/indexed HTML does not exist.

Never send a password through `update_foleo_artifact_settings`; the tool
refuses it (`password_requires_dashboard`). The human sets it in the Foleo
dashboard so the secret never passes through an agent transcript.
