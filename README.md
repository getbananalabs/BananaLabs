```
██████╗  █████╗ ███╗   ██╗ █████╗ ███╗   ██╗ █████╗
██╔══██╗██╔══██╗████╗  ██║██╔══██╗████╗  ██║██╔══██╗
██████╔╝███████║██╔██╗ ██║███████║██╔██╗ ██║███████║
██╔══██╗██╔══██║██║╚██╗██║██╔══██║██║╚██╗██║██╔══██║
██████╔╝██║  ██║██║ ╚████║██║  ██║██║ ╚████║██║  ██║
╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝
██╗      █████╗ ██████╗ ███████╗
██║     ██╔══██╗██╔══██╗██╔════╝
██║     ███████║██████╔╝███████╗
██║     ██╔══██║██╔══██╗╚════██║
███████╗██║  ██║██████╔╝███████║██╗
╚══════╝╚═╝  ╚═╝╚═════╝ ╚══════╝╚═╝

      Tools for Claude Code from Banana Labs.
```

This repository is a Claude Code plugin marketplace. Add it once, then
install the plugins you want. Anyone can add it: the repository is public.

```bash
claude plugin marketplace add getbananalabs/BananaLabs
```

| plugin | what it does | install |
|---|---|---|
| **BananaMigrator.** | Move a site off Framer onto your own hosting: an exact copy, the project source, 13 checks that the copy matches the original, form flows, snapshots and change reports. | `claude plugin install bananamigrator@bananalabs` |

Each plugin lives in `plugins/<name>/` with its own README. Skills are
namespaced under the plugin's name, so the BananaMigrator. skill runs as
`/bananamigrator:migrate`.

## How a site travels with BananaMigrator.

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

The copy is pulled from the published site, the source from the Framer
project, and `verify` compares the copy with the original with 13 checks at
3 widths before and after deployment. The plugin's skill walks a Claude Code
session through the whole procedure and reads the tool's reports.

## Updates

Auto-update is off by default for a marketplace you added yourself. Turn it
on under **Marketplaces** in `/plugin`, or run
`claude plugin update bananamigrator@bananalabs` when you want the latest
version.

## License

MIT. See `LICENSE`.
