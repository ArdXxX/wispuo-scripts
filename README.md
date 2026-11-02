# WispUO script library

The official library of scripts for [WispUO](https://github.com/ArdXxX/WispUO), the Ultima Online client without graphics driven by
scripts. WispUO's script editor has a **Library** tab that reads this repository: it lists the scripts, shows their code, and
installs the ones the user chooses. **Nothing is ever run, opened or updated by itself.**

This repository is currently empty of scripts: the structure and the rules below are what contributions follow.

## Layout

```
README.md
.gitattributes            "* -text": no line-ending conversion, so the bytes on disk are the bytes GitHub serves
index.json                generated; lists every script with its SHA-256 (see "Building the index")
scripts/
  common/                 scripts for any shard
    <folders>/Script.oajs
    lib/Helpers.oajs      include-only: "lib" folders are not listed, only installed with the scripts that include them
  official/               one folder per shard; the folder name is the shard id
    shard.json            { "name": "...", "hosts": ["login address", ...] }
  <shard-id>/
    shard.json
    Mining/Miner.oajs
```

WispUO recognises the shard of an account by the **login address** of its server (`hosts` in `shard.json`), and shows that
shard's folder together with `common`.

## Script header

A script may start with a header that the index builder reads (WispUO itself only reads `index.json`):

```js
// @name        Miner
// @description Mines ore around the character and smelts it.
// @author      Someone
// @version     1.2
// @updated     2026-10-30
// @hidden      (optional: include-only, not listed)
```

Includes use the usual `//#include relative/path.oajs`; the builder follows them (transitively) and records them in the index.

## `index.json`, format 1

```json
{
  "format": 1,
  "shards": [ { "id": "official", "name": "Ultima Online (official, EA/OSI)", "hosts": ["75.2.70.184"] } ],
  "scripts": [
    {
      "path": "scripts/common/Examples/HelloWorld.oajs",
      "name": "Hello world",
      "description": "...",
      "author": "...",
      "version": "1.0",
      "updated": "",
      "sha256": "<64 hex digits of the exact bytes>",
      "size": 247,
      "listed": true,
      "includes": ["scripts/common/lib/Messages.oajs"]
    }
  ]
}
```

Rules (an entry that breaks one is skipped by the client, which says why):

- `format` must be `1`; another number rejects the whole file and the client keeps its previous copy.
- `shards[].id`: lowercase slug `a-z 0-9 -`, at most 40 characters, not `common`, unique; it is the folder name under `scripts/`.
- `size`: 1 to 524,288 bytes (512 KB) per file; a script with all its includes at most 4 MB; at most 32 includes; the whole index
  at most 1 MB and 5,000 entries.
- `includes` lists every library path included, transitively, sorted; each must itself be in the index.

## Library paths

`scripts/<shard>/<folders…>/<file>`, written exactly so (anything else is refused, never "fixed"):

- `/` separators only, at most 160 characters, between 3 and 8 segments; segment 1 is `scripts`, segment 2 is `common` or a shard id.
- Other segments: 1-64 characters of `A-Z a-z 0-9 space _ - . ( )`, ASCII only, not starting with a space or `.`, not ending with
  `.` or a space, not a Windows device name (`CON`, `NUL`, `COM1`, …).
- The file ends in `.oajs` or `.js`. No `.oajsp`/`.jsp` (protected scripts cannot be shown before use) and no other file types.

## What users see before installing

- The code of **every** file the script needs (the script and its includes) is shown for review before anything is written.
  *Install* is never the default button.
- Each file is checked against `index.json` (SHA-256 and size) before being written; a mismatch writes nothing.
- Warnings are shown for sensitive APIs: network, launching programs, files, `eval`, Autostart.
- Everything is downloaded over https from this repository only, through the account's own proxy when it has one.

## Security summary

- Scripts can do anything a WispUO script can do. Read them before installing. The hash protects against corruption in transit,
  not against a compromised repository; signing the index is planned.
- Contributions are reviewed by hand. Do not add scripts that steal credentials, call out to unknown hosts, or hide what they do.

## Contributing

1. Put the script under `scripts/common/…` (any shard) or `scripts/<shard-id>/…` (one shard); add `shard.json` for a new shard.
2. Add the header, keep the file ASCII-friendly UTF-8 with LF line endings.
3. Rebuild the index with the tool in the WispUO repository and commit `index.json` with the script:

```bat
dotnet run --project tools\script-library -- build <path to this repository>
```

## Publishing

This repository is a copy of the `script-library/` folder of the WispUO repository; keep both in step.
The index address WispUO uses is
`https://raw.githubusercontent.com/ArdXxX/wispuo-scripts/main/index.json` (changeable in Help → Panel settings). If GitHub's
limits on anonymous raw downloads become a problem, the same repository can be served from GitHub Pages by changing only that
address.
