# Active HTML artifacts

Active HTML is executable: it runs JavaScript, shows forms, and can start
downloads. Foleo hosts it only for explicitly enrolled accounts. The human
sets each connection's publishing policy in Foleo; the agent cannot change it.

## Before anything else

Check `get_foleo_capabilities`. If `availability.namedAlphaHtml` is false, this
account cannot publish active HTML — say so and stop. Calling the tools anyway
returns `html_unavailable`. Enrollment is an operator decision, never something
to ask the server for twice.

The new-connection default is `private`; existing grants keep `ask_each_time`
until the human changes them in Foleo Access. `private` permits clean private
pages to publish in one call. The human may opt into
`private_and_contained_unlisted`: a clean unlisted page that fits the contained
runtime profile can then publish in one call. Open unlisted pages still need
per-candidate approval. Anyone with an unlisted URL can read it.

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

Publishing HTML over MCP stages exact bytes first. A covered private page with
a clear safety gate can then publish in that same call. A safety match needs
human review.

1. Call `publish_foleo_artifact` with an inline single-file page:

   ```json
   {
     "source": { "sourceFormat": "html", "content": "<!doctype html>…" },
     "name": "<label>",
     "idempotencyKey": "<uuid>"
   }
   ```

   If the result says `published`, give the human its `url` and explain its
   `access`, `runtimeProfile`, `profileNote` and undo. If it says `approval_required`, nothing
   is live: the server staged the exact bytes and returned an `approvalId`.

2. For `approval_required`, give the human the `reviewUrl`. They see the exact source and its hash on
   a Foleo page — never on the artifact's own domain — and approve or reject.
   `get_foleo_publish_approval` tells you where it stands; poll it politely,
   and never pretend an approval happened.

3. Once approved, call `commit_foleo_publish` with the `approvalId` and a stable
   `idempotencyKey`. Only now does the artifact exist.

An approval is bound to this connection, organization, operation, content
revision, security settings and exact content hash. A title-only edit can
survive; content, access, name, status, policy or eligibility drift cannot.
An approval expires after 30 minutes. Within 24 hours of creation, an expired
candidate can be restaged from retained bytes with `restageFromApprovalId` and
an `idempotencyKey` alone. Never send content or changed settings alongside it.

Multi-page sites and local asset bundles are not an MCP operation. That is what
the Foleo CLI is for; say so instead of inlining a build.

## Names are the address

An active-HTML artifact lives on its own subdomain. `name` may be omitted;
Foleo derives one from `settings.title` or the page's `<title>` and returns it.
Rules: 3–63 characters, `a-z`, `0-9`, and hyphens, with no leading, trailing,
or doubled hyphen.

Renaming later is an explicit decision about the old URL, and it is the
human's to make, not yours:

- `rename_foleo_artifact_name` with `previousLabel: "keep-as-alias"` keeps the
  old URL working.
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
