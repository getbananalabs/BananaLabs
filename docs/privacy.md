# Privacy policy

This policy covers the BananaMigrator. command line tool and Claude Code
plugin published by Banana Labs, and the pages that describe them.

## What the tool does with data

BananaMigrator. runs on your machine. It reads:

- the published website you point it at,
- your Framer project through the Framer Server API, with the API key you
  put into your own `.env` file, using read methods only,
- the hosting where you deploy the copy, when you verify it there,
- your own S3-compatible storage (such as Cloudflare R2), only when you run a
  snapshot with `--push` and only with the credentials you configured.

It writes the copy, the project source, reports and snapshots into the
`out/` directory of your checkout. Form flows send the test values you wrote
into your site configuration to your own forms endpoint and, if you ask for
it with `original: true`, to Framer.

Nothing is sent to Banana Labs. The tool has no telemetry, no analytics, no
account and no update check of its own.

## What the plugin does with data

The plugin adds a skill to Claude Code. The skill is text that Claude reads;
it collects nothing and calls no service of ours. Anything the skill asks
Claude Code to run goes through the tool described above, on your machine.

## Keys and secrets

Your Framer Server API key and storage credentials live in your `.env` file,
which the tool never copies, logs or reports. Keep it out of version control
and out of support requests.

## These pages and support requests

These pages are hosted on GitHub, so GitHub's privacy statement applies to
visiting them. Support requests go through our helpdesk at
https://helpdesk.twogether.one/submit; the details you send there (your name,
email address and the content of your request) are used only to answer you.

## Contact

Questions about this policy: https://helpdesk.twogether.one/submit.

Last updated: 8 October 2026.
