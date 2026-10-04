# stash-scraper-repo

A Stash scrapers source index: the scrapers live in `scrapers/`, and every push
to `main` builds them into an index that Stash installs from.

Source index URL:

```
https://ravenddddd.github.io/stash-scraper-repo/main/index.yml
```

## What is in here

| Package | Scrapers | Covers |
|---|---|---|
| `DMM-ravend` | `DMM-ravend` (scenes), `DMM-ravend-doujin` (galleries), `DMM-ravend-book` (galleries) | `www.dmm.co.jp` mono, `dmm.co.jp/dc/doujin`, `book.dmm.co.jp` |
| `DLsite-ravend` | `DLsite-ravend` | `dlsite.com` work pages |

The `-ravend` suffix keeps these ids from colliding with the community
repository's `DMM` and `DLsite`. **Stash keys a scraper by its filename**, so two
sources shipping a `DMM.yml` fight over the same id, and which one wins depends
on directory walk order. Renaming the files is what keeps both installable —
and the `name:` fields carry the suffix too so the UI does not list two
scrapers called "DMM".

## Where this came from

- The repository scaffolding is a fork of
  [stashapp/scrapers-repo-template](https://github.com/stashapp/scrapers-repo-template).
- The build and validation tooling (`build.ts`, `validate.js`, `lib/`,
  `validator/`) is copied from
  [stashapp/CommunityScrapers](https://github.com/stashapp/CommunityScrapers),
  which is also where the `DMM` and `DLsite` scrapers these are derived from
  live. The copied tooling replaced the template's `build_site.sh`, which does
  not skip the scrapers inside a package folder and so would emit duplicate
  index entries for `DMM-ravend`.
- The `DMM` and `DLsite` scrapers here are modified versions of the community
  ones. To see what upstream changed since the last sync, add it as a remote:

  ```bash
  git remote add official https://github.com/stashapp/CommunityScrapers.git
  git fetch official
  git diff official/master -- scrapers/DMM/DMM-doujin.yml
  ```

  Do not `git checkout official/master -- <path>` on these files — that would
  overwrite the local changes, which is the whole point of the fork.

## Installing in Stash

Add this as a second package source under **Settings → Metadata Providers →
Scrapers → Package Sources**. The community source can stay:

```yaml
scrapers:
  package_sources:
    - name: Community (stable)
      url: https://stashapp.github.io/CommunityScrapers/stable/index.yml
      localpath: community
    - name: ravenddddd
      url: https://ravenddddd.github.io/stash-scraper-repo/main/index.yml
      localpath: ravenddddd
```

## Working on it

Scrapers are plain YAML in `scrapers/`. A folder containing a `package` file is
published as a single package; a scraper file outside such a folder is its own
package.

```bash
deno -R -E ./validate.js      # check scrapers against the schema
deno task build _site         # build the index locally
```

Both run in CI: `validate.yml` on every push and pull request, and again inside
`deploy.yml` before publishing, so a scraper that fails validation cannot reach
the index.

Note that Stash loads scraper YAML **strictly** at the top level, but the
gallery and scene sub-configs are unmarshalled a second time without that
strictness — so a misspelled field inside a `gallery:` block is silently
ignored rather than reported. The schema validation above is what catches it;
Stash will not.

## Licence

AGPL-3.0. The build and validation tooling copied from CommunityScrapers is
AGPL-3.0, and so is the template this repository forks.
