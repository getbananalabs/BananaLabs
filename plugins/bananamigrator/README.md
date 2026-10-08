```
        ██████╗  █████╗ ███╗   ██╗ █████╗ ███╗   ██╗ █████╗
        ██╔══██╗██╔══██╗████╗  ██║██╔══██╗████╗  ██║██╔══██╗
        ██████╔╝███████║██╔██╗ ██║███████║██╔██╗ ██║███████║
        ██╔══██╗██╔══██║██║╚██╗██║██╔══██║██║╚██╗██║██╔══██║
        ██████╔╝██║  ██║██║ ╚████║██║  ██║██║ ╚████║██║  ██║
        ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝
███╗   ███╗██╗ ██████╗ ██████╗  █████╗ ████████╗ ██████╗ ██████╗
████╗ ████║██║██╔════╝ ██╔══██╗██╔══██╗╚══██╔══╝██╔═══██╗██╔══██╗
██╔████╔██║██║██║  ███╗██████╔╝███████║   ██║   ██║   ██║██████╔╝
██║╚██╔╝██║██║██║   ██║██╔══██╗██╔══██║   ██║   ██║   ██║██╔══██╗
██║ ╚═╝ ██║██║╚██████╔╝██║  ██║██║  ██║   ██║   ╚██████╔╝██║  ██║██╗
╚═╝     ╚═╝╚═╝ ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝    ╚═════╝ ╚═╝  ╚═╝╚═╝

   Move a site off Framer: exact copy, source, machine-verified.
```

BananaMigrator. moves a site off Framer onto your own hosting. It pulls the
published site as an exact copy, extracts the project source through the
Framer Server API and verifies by machine that the copy is 1:1 with the
original: 13 checks at 3 widths, form flows run by the site's own client
code, snapshots and change reports. The site keeps being edited in Framer and
can be pulled again at any time.

This plugin adds the skill `/bananamigrator:migrate`, which walks a Claude
Code session through the whole procedure, from the site configuration to the
verification before and after the DNS cutover, and reads the tool's reports
and findings for you.

## Install

```bash
claude plugin marketplace add getbananalabs/BananaLabs
```

```bash
claude plugin install bananamigrator@bananalabs
```

## How a site travels

```
 Framer                     BananaMigrator.               your hosting
 ──────                     ───────────────               ────────────

 published site ──mirror──▶ out/<site>/mirror ──export──▶ target site
        │                   pages, public,                (repo, hosting)
        │                   manifest.json                      │
        │                           │                          │
        │                         serve                        │
        │                     local preview                    │
        │                           │                          │
        └────── original ─────▶  verify  ◀──────── copy ───────┘
                          13 checks at 3 widths
                       --flows, --resolve host=ip
                                    │
                               PASS / FAIL ──▶ DNS cutover

 Server API project ─dump─▶ out/<site>/dump
                            code, CMS, styles, node tree
```

## The tool

The skill drives the BananaMigrator. command line tool, which lives in its own
repository: https://github.com/getbananalabs/BananaMigrator. It needs Node 22,
pnpm and Playwright's Chromium, plus a Framer Server API key for the project
(paid Framer plan) when you want the project source as well as the copy.
