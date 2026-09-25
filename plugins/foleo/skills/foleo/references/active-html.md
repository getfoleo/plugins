# Active HTML artifacts

Active HTML is executable: it runs JavaScript, shows forms, and can start
downloads. Foleo hosts it only for explicitly enrolled accounts, and only with
a human approving the exact bytes.

## Before anything else

Check `get_foleo_capabilities`. If `availability.namedAlphaHtml` is false, this
account cannot publish active HTML — say so and stop. Calling the tools anyway
returns `html_unavailable`. Enrollment is an operator decision, never something
to ask the server for twice.

Then get the human's approval for the hosting risk itself, separately from
approving the content: hosted HTML runs in the reader's browser on a Foleo
subdomain. Tell them to use no sensitive information and to open test links in
a dedicated browser profile.

## The approval loop

Publishing HTML over MCP is deliberately two-step, and the human stands in the
middle of it.

1. Call `publish_foleo_artifact` with an inline single-file page:

   ```json
   {
     "source": { "sourceFormat": "html", "content": "<!doctype html>…" },
     "name": "<label>",
     "idempotencyKey": "<uuid>"
   }
   ```

   Nothing is published. The server stages the exact bytes and returns an
   approval record with an `approvalId`.

2. Give the human the `reviewUrl` from that response. They see the exact source and its hash on
   a Foleo page — never on the artifact's own domain — and approve or reject.
   `get_foleo_publish_approval` tells you where it stands; poll it politely,
   and never pretend an approval happened.

3. Once approved, call `commit_foleo_publish` with the `approvalId` and a fresh
   `idempotencyKey`. Only now does the artifact exist.

An approval is bound to this connection, this organization, this operation,
this artifact revision, and that exact content hash. Changing a single byte
invalidates it — go back to step 1 rather than editing after approval. It also
expires, so do not stage a candidate hours before the human is available.

Multi-page sites and local asset bundles are not an MCP operation. That is what
the Foleo CLI is for; say so instead of inlining a build.

## Names are the address

An active-HTML artifact lives on its own subdomain, so `name` is required at
creation — there is no generated fallback host. Choose it with the human.
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
