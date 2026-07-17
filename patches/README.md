# Deacon Social patches

These patches are applied on top of each upstream Mastodon stable release by
the *Track upstream releases* workflow (`.github/workflows/upstream-sync.yml`).
When a new upstream release appears, it applies `patches/`, swaps upstream's
workflow files for the fork's own, tags the result `vX.Y.Z-deacon.1`, and
pushes the tag over SSH using the `GH_DEPLOY_KEY` secret (a write deploy key —
`GITHUB_TOKEN` is not allowed to push upstream's workflow-touching history).
The tag push triggers the *Build images* workflow
(`.github/workflows/build-image.yml`), which publishes
`ghcr.io/deaconsocial/mastodon` and `ghcr.io/deaconsocial/mastodon-streaming`
tagged `vX.Y.Z-deacon.1`.

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

# Mirror what the sync workflow does: fork workflows, then the tag
git rm -r .github/workflows
git checkout main -- .github/workflows
git commit -m "Replace upstream workflow files with fork workflows"
git tag vX.Y.Z-deacon.1
git push origin vX.Y.Z-deacon.1        # this push triggers the image build

# Regenerate the patch files and commit them on the default branch
git format-patch "vX.Y.Z..vX.Y.Z-deacon.1~1" -o /tmp/patches-new
git checkout main
rm patches/*.patch && cp /tmp/patches-new/*.patch patches/
git commit -am "Refresh patches for vX.Y.Z" && git push
```

(The `~1` in the `format-patch` range excludes the workflow-swap commit, which
must not become a patch.)
