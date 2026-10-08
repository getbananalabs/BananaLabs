# BananaMigrator. documentation

BananaMigrator. moves a website off Framer onto your own hosting. It pulls an
exact copy of the published site, extracts the project source through the
Framer Server API and verifies by machine that the copy is 1:1 with the
original. The site keeps being edited in Framer and can be pulled again at
any time. Nothing is deployed, published or written to Framer by the tool.

This page is the user guide. The plugin's skill (`/bananamigrator:migrate`)
walks a Claude Code session through the same procedure and reads the tool's
reports for you.

## What you get

| | |
|---|---|
| **Mirror** | an exact copy of the published site: pages, runtime modules, images with all variants, fonts, CMS data, icons, robots, sitemap, the 404 page |
| **Dump** | the project source through the Server API: code files (TSX), CMS, styles, the node tree of every page and component with its texts, redirects |
| **Verify** | 13 checks at 3 widths that the copy matches the original, plus form flows run by the site's own client code |
| **Export** | hands the copy to the repository of the target site |
| **Snapshots** | the site as of a date, locally or in S3-compatible storage, and a report of what changed since the last pull |

## Requirements

- Node 22 or newer and pnpm.
- Chromium for Playwright: `pnpm exec playwright install chromium`.
- For the dump: a Framer Server API key for the project (Settings → General →
  API keys in the project; available on paid Framer plans). The mirror and
  verify work without it.
- Claude Code, if you want the skill. The tool works on its own as well.

## Install

The plugin, from any directory:

```bash
claude plugin marketplace add getbananalabs/BananaLabs
```

```bash
claude plugin install bananamigrator@bananalabs
```

The tool:

```bash
git clone https://github.com/getbananalabs/BananaMigrator.git
```

```bash
cd BananaMigrator && pnpm install && pnpm exec playwright install chromium
```

## Configure a site

Each site has a `sites/<site>.json`:

```json
{
  "url": "https://www.example.com",
  "projectUrl": "https://framer.com/projects/…",
  "apiKeyEnv": "FRAMER_API_KEY_EXAMPLE",
  "formsEndpoint": "/api/framer-forms",
  "allowedThirdPartyHosts": ["www.googletagmanager.com", "challenges.cloudflare.com"],
  "discoverPaths": ["/#order?package=1"],
  "flows": [
    {
      "name": "contact",
      "path": "/contact",
      "anchorField": "Email",
      "fill": { "Name": "Test", "Email": "test@example.com", "Message": "Test submission, please ignore." },
      "expectRequest": "/api/framer-forms/",
      "expectText": "Thank you"
    }
  ]
}
```

- `url`: the published address. The mirror sees only what is published.
- `projectUrl` and `apiKeyEnv`: the project in the Framer editor and the name
  of the environment variable that holds its Server API key. The key goes
  into `.env`, never into the configuration.
- `formsEndpoint`: where the forms in your copy post to. Forms are rebuilt on
  your side; this makes sure nothing leaves for api.framer.com.
- `allowedThirdPartyHosts`: hosts the copy may call (analytics, Turnstile,
  your own Worker). Anything else is reported.
- `discoverPaths`: extra paths for the runtime pass, typically ones that open
  a popup loaded on demand.
- `flows`: scenarios for `verify --flows`. A flow starts at `path` and runs
  `steps` (`click`, `waitFor`, `wait`, `goto`, `press`, `fill`, `select`,
  `check`, `submit`), or the shortcut `fill` plus submit. Expectations:
  `expectRequest` with `expectStatus` (default `2xx`), `expectText`,
  `expectUrl`. `original: true` runs the flow against the original too.

## Procedure

1. **Pull.** `pnpm pull <site>` fetches the mirror and the dump. The text of
   every published page is completed from the mirror, because the Server API
   reads text only from the page the project has open.
2. **Verify locally.** `pnpm verify <site>` serves the copy on localhost and
   compares it with the original. Anything other than `PASS` is fixed in the
   configuration or the tool, never by hand in the copy.
3. **Rehearse the forms.** `pnpm verify <site> --flows-only`. The local
   preview answers the forms endpoint with 200 and sends nothing. Whatever
   the copy fails to serve during a flow (an icon loaded only after a
   submit) is reported, written into the mirror's manifest and fetched by
   the next `pnpm mirror <site> --no-discover`.
4. **Hand over.** `pnpm export <site> --to <repo>/sources/<site>/mirror`. The
   target site serves `pages/*.html` by path, `public/` at the domain root,
   replaces `https://__ORIGIN__` in text files with its own origin, takes
   content types from the manifest, allows the third-party hosts and
   `worker-src blob:` in its CSP, and provides the forms endpoint.
5. **Verify the deployment.** `pnpm verify <site> --copy https://… --flows`.
   Before the DNS cutover, under the real host name:
   `--resolve www.example.com=<ip>`. Flows send real data at this point.
6. **DNS cutover**, by you. Afterwards the original lives at the project's
   `*.framer.app` address: pass it with `--original`.
7. **Keep up.** `pnpm pull <site> --snapshot` on later pulls and
   `pnpm diff <site>` to see what changed: pages, files, texts, CMS, code,
   styles, redirects. `--push` uploads a snapshot to Cloudflare R2 or any
   S3-compatible storage configured in `.env`.

## The 13 checks

| # | check | what it guards |
|---|---|---|
| 1 | element tree | tag, classes, 27 attributes and the text of every element, in the same order |
| 2 | computed styles | 52 properties of every element: typography, colours, dimensions, layout, backgrounds, shadows, filters |
| 3 | geometry | position and size of every element, within 1 px |
| 4 | head | title, description, robots, og and twitter meta, canonical, icons, alternate |
| 5 | status code | the original and the copy return the page with the same HTTP status |
| 6 | console | no errors in the copy's console |
| 7 | requests | no 4xx/5xx responses and no failed requests from the copy |
| 8 | third-party hosts | the copy only calls its own domain and the allowed hosts |
| 9 | internal links | every link on the copy to its own domain resolves |
| 10 | unknown path | returns 404 |
| 11 | robots.txt | identical to the original |
| 12 | sitemap.xml | identical to the original |
| 13 | flows | the expected request and status, the success text, the final address |

Checks 1 to 9 run for every page at every width (desktop 1440, tablet 1000,
phone 390), 10 to 12 once per site, 13 on request. The result is
`out/<site>/verify/report.json` plus a summary on the console; a single
failure means exit code 1.

## Commands

| command | what it does |
|---|---|
| `pnpm mirror <site> [--refresh] [--no-discover]` | exact copy of the published site |
| `pnpm serve <site> [--port 4310]` | local preview of the copy |
| `pnpm verify <site> [--copy https://…] [--original https://…] [--pages /a,/b] [--viewports desktop,tablet,phone] [--resolve host=ip] [--flows [names]] [--flows-only]` | the checks and flows |
| `pnpm dump <site>` | the project source through the Server API |
| `pnpm export <site> --to <dir>` | hand the copy to the target site |
| `pnpm pull <site> [--snapshot [--push]]` | mirror + dump, optionally a snapshot |
| `pnpm snapshot <site> [--push]`, `pnpm snapshots <site>`, `pnpm diff <site> [a] [b]` | snapshots and their comparison |

## Limits

- Drafts aren't published, so they aren't in the mirror and keep no text in the dump.
- Text nodes inside component masters stay without text; the instances on the pages have theirs.
- The copy is a copy of the published output. Framer's runtime keeps working
  in it; the tool doesn't rebuild the site without the runtime.
- Framer's analytics and editor bar are removed from the copy. The site's own
  Google Analytics stays.

## Where data goes

Everything runs on your machine. The tool reads your published site, your
Framer project (with your own key, read methods only) and the hosting you
point it at, and writes into `out/<site>/` in your checkout. Snapshots leave
your machine only when you run `--push` with your own storage credentials.
Nothing is sent to Banana Labs. See the [privacy policy](privacy.md).

## Support

Report a problem or ask a question at https://helpdesk.twogether.one/submit.
Include the tool version (from `package.json`), the site name, the command
you ran, the console output and, when it exists, `out/<site>/verify/report.json`
or `flows.json`. Never send your `.env`.
