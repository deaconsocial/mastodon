# Deacon Social patches

These patches are applied on top of each upstream Mastodon stable release by
the *Track upstream releases* workflow (`.github/workflows/upstream-sync.yml`),
which then tags the result `vX.Y.Z-deacon.1` and dispatches the *Build images*
workflow to publish `ghcr.io/deaconsocial/mastodon` and
`ghcr.io/deaconsocial/mastodon-streaming` for that tag.

## Current patches

- `0001-Increase-status-character-limit-to-10-000.patch` — raises the status
  character limit from 500 to 10,000 (server-side validator and web composer
  fallback).

## Refreshing a patch after a conflict

When the sync workflow opens a "Manual rebase needed" issue, the patch no
longer applies to the new upstream release. To refresh it:

```sh
git fetch upstream --tags
git checkout --detach vX.Y.Z           # the new upstream tag
git am --3way patches/*.patch          # fix conflicts, then: git am --continue
git tag vX.Y.Z-deacon.1
git push origin vX.Y.Z-deacon.1

# Regenerate the patch files and commit them on the default branch
git format-patch -1 vX.Y.Z-deacon.1 -o /tmp/patches-new
git checkout main
rm patches/0001-*.patch && cp /tmp/patches-new/*.patch patches/
git commit -am "Refresh patches for vX.Y.Z" && git push
```

Then run the *Build images* workflow manually (Actions → Build images →
Run workflow) with `ref` set to `vX.Y.Z-deacon.1`.
