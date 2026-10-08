---
name: migrate
description: Moving a site off Framer with BananaMigrator. (mirror, dump, verify, flows, export, snapshot, diff) – which command when, how to read the reports and findings, what the tool deliberately doesn't do. Use it when someone wants to move, pull or mirror a Framer site, verify that a copy matches the original, rehearse forms on the copy, or find out what changed on the site since the last pull.
argument-hint: "<site> [step]"
---

# BananaMigrator. – moving a site off Framer

The tool pulls a published Framer site as an exact copy, extracts the
project source through the Server API and verifies by machine that the copy
is 1:1 with the original. The site keeps being edited in Framer and can be
pulled again at any time. `verify` decides, not a pair of eyes. The tool
**deploys nothing, publishes nothing and never writes to Framer**.

Details and numbers are in `README.md` (English) and `README.cs.md` (Czech),
the working rules in `CLAUDE.md`. This skill is the procedure and how to read
the results.

## Where things are

| path | what's in it |
|---|---|
| `sites/<site>.json` | site configuration: `url`, `projectUrl`, `apiKeyEnv`, `formsEndpoint`, `allowedThirdPartyHosts`, `discoverPaths`, `flows` |
| `.env` | the Server API key (`FRAMER_API_KEY_<SITE>`) and snapshot storage access; never in the configuration, a log or a chat |
| `out/<site>/mirror/` | the copy: `pages/*.html`, `public/…`, `manifest.json` (pages, files, stripped things, forms, CMS, runtime findings) |
| `out/<site>/dump/` | the source: `project.json`, `styles.json`, `code/`, `cms/`, `tree.json` (node tree with texts) |
| `out/<site>/verify/` | `report.json` from a full verification, `flows.json` from a flows-only run |
| `out/<site>/snapshots/` | snapshots `<date>/` with a `snapshot.json` summary, `diff.json` from the last comparison |

Binary files aren't versioned; manifests, reports, the dump and snapshot summaries are.

## Procedure

Every step has a condition you don't move on without.

1. **Site configuration.** Create `sites/<site>.json` following the example in
   the README. The Server API key (Settings → General → API keys in the
   project, paid plan) goes into `.env` under the name from `apiKeyEnv`. Put
   only hosts the site really calls into `allowedThirdPartyHosts` (analytics,
   Turnstile, your own Worker).
2. **Pull.** `pnpm pull <site>` = `mirror` (two passes: static, and runtime in
   Chromium) + `dump` (project, styles, code, CMS, node tree; the text of every
   published page is completed from the mirror). Condition: the log shows
   `unmatched 0` for texts and no `leftover Framer references` that would load.
3. **Verify locally.** `pnpm verify <site>` serves the copy locally and
   compares it with the original at 3 widths with 13 checks. Condition: `PASS`.
   Anything else is fixed in the tool or the configuration, never by hand in the copy.
4. **Flows locally.** `pnpm verify <site> --flows-only`: forms and click paths
   run by the site's own client code. The preview answers `formsEndpoint` with
   200 and sends nothing. Condition: every flow `OK`. `missing in copy: …`
   means an asset loaded only after the submit; verify wrote it into the
   manifest, `pnpm mirror <site> --no-discover` fetches it, run the flow again.
5. **Hand over to the target site.** `pnpm export <site> --to <repo>/sources/<site>/mirror`.
   The target site serves `pages/*.html` by path (an unknown path gets
   `404.html` with status 404), `public/` at the domain root, fills
   `https://__ORIGIN__` in text files with its own origin, takes content types
   from the manifest, allows `allowedThirdPartyHosts` and `worker-src blob:`
   in its CSP (the Framer form client), and has the `formsEndpoint` the forms post to.
6. **Verify the deployed copy.** `pnpm verify <site> --copy https://… --flows`.
   Before the DNS cutover, under the real host: `--resolve www.example.com=<ip>`
   (TLS errors are tolerated, the original and the copy run in separate
   browsers). Condition: `PASS` including flows. Flows send real data.
7. **DNS cutover** is done by a person. Afterwards the original lives at the
   project's `*.framer.app` address: `pnpm verify <site> --copy https://www.… --original https://<project>.framer.app`.
8. **Snapshot.** `pnpm snapshot <site>` (with `--push` to R2/S3, access in
   `.env`). On later pulls `pnpm pull <site> --snapshot` and `pnpm diff <site>`
   says what changed on the site: pages, files, texts, CMS, code, styles.

## How to read the results

| message | meaning | what to do |
|---|---|---|
| `FAIL desktop /path … diffs N` | the tree, a style or an element's rectangle differs | open `report.json`, field `diffs`: element path, field, original, copy; the cause is usually a missing file or a rewritten address |
| `head N` | title, description, canonical, og or icons differ | compare `headDiffs`; often a canonical with another host |
| `third-party hosts N` | the copy calls a host outside `allowedThirdPartyHosts` | either allow it (it belongs to the site), or it's a leak to Framer and a rewrite is missing |
| `requests N` | a 4xx/5xx response or a failed request on the copy | `badRequests` has the address; missing in the mirror, or the target site doesn't know the content type |
| `links N` | an internal link on the copy doesn't exist | the page isn't published, or the path is missing from the sitemap |
| `unmatched N` (dump, texts) | a text node of the tree wasn't found in the HTML | the node is hidden, in a draft or outside the breakpoints; check in `tree.json` |
| `unpublished changes N` (dump) | the text in the editor differs from the published one | the node has `publishedText`; the site in Framer is waiting for a publish |
| `not in the mirror (draft)` | the page isn't published | nothing, drafts don't move |
| `missing in copy: /m/…` (flow) | an asset loaded only after the submit | `pnpm mirror <site> --no-discover`, run the flow again |
| `no request to … within N s` (flow) | the submit triggered no request | the form didn't hydrate (modules missing), or the CSP blocks the client's Worker |
| `response 4xx` (flow) | the forms endpoint on the target site | check `formsEndpoint` and the deployed endpoint, not the tool |

Exit code 1 = something failed. The report is always in `verify/report.json`
or `verify/flows.json`; the console summary is abbreviated.

## Findings from practice

What verify caught on the first site moved and nobody would have found by
eye: a module that crashed on a relative path and made the runtime switch off
the whole UI; Framer's editor bar calling framer.com for every visitor; icons
assembled at runtime (`framer.com/m/…`); a gallery left empty because the
code recognises a Framer image by the word `framerusercontent` in the
address; a CSP without `worker-src blob:` that stopped forms from being
submitted; a newsletter success icon loaded only after the submit. When you
see something like this, the fix belongs in the tool or the configuration,
so every next site gets it.

## Rules

- A push goes out only on the owner's explicit word, one word = one push.
  Commit locally as often as you like; before a push, say what the batch holds.
- Server API keys and storage access live only in `.env`. Never in a site's
  configuration, a log, a report or a chat. The dump calls only read methods.
- `verify` decides, not a look in the browser. When you want to "check"
  something, write a check or a flow.
- The README is bilingual (`README.md`, `README.cs.md`) and both change
  together; this skill is the third place that has to match the CLI.
- The version lives in `package.json` and in the commit subject
  (`v0.14.0: …`): a fix = patch, a new capability = minor.
- The product name is **BananaMigrator.** with the dot, mid-sentence too.
  Texts use a short dash, never a long dash and never a middle dot.

## Commands

| command | what it does |
|---|---|
| `pnpm mirror <site> [--refresh] [--no-discover]` | exact copy of the published site |
| `pnpm serve <site> [--port 4310]` | local preview of the copy, forms get 200 without sending |
| `pnpm verify <site> [--copy https://…] [--original https://…] [--pages /a,/b] [--viewports desktop,tablet,phone] [--resolve host=ip] [--flows [names]] [--flows-only]` | 13 checks and flows |
| `pnpm dump <site>` | the project source through the Server API, texts from the mirror |
| `pnpm export <site> --to <dir>` | hand the copy to the target site |
| `pnpm pull <site> [--snapshot [--push]]` | mirror + dump, optionally a snapshot |
| `pnpm snapshot <site> [--push]`, `pnpm snapshots <site>`, `pnpm diff <site> [a] [b]` | snapshots and their comparison |
