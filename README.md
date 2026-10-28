# renovate-config

Shared Renovate presets for yuanying's repositories.

## Usage

In a repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>yuanying/renovate-config",
    "github>yuanying/renovate-config:automerge-patch"
  ]
}
```

`github>yuanying/renovate-config` is the default preset (`default.json`) and is
meant for every repository. `github>yuanying/renovate-config:automerge-patch`
is opt-in; leave it out where every merge is left to a person.

## Presets

### `default.json`

- Extends `config:recommended` and `:semanticCommitTypeAll(chore)`: in a
  repository whose history Renovate reads as using semantic commits, every
  update is a `chore(deps):` commit. Elsewhere the commits keep Renovate's
  plain `Update ...` subjects.
- `timezone` is `Asia/Tokyo` and the Dependency Dashboard is on.
- `minimumReleaseAge` is `2 days`, with `internalChecksFilter: strict`: an
  update gets no branch until it has been public for two days.
- `platformAutomerge` is `false`: whatever Renovate is allowed to merge, it
  merges itself once it sees the checks green, without the repository's
  auto-merge setting or branch protection.
- The official `ubuntu` image from Docker Hub has no wait. Docker Hub gives a
  tag's last push as its release time, and the ubuntu tags are pushed again
  every day or two, so with the wait an update would hardly ever be proposed.
- Everything the `github-actions` manager reads is grouped into one
  `github-actions` branch.

### `automerge-patch.json`

- Patch, digest and pinning updates (`patch`, `digest`, `pin`, `pinDigest`)
  are merged by Renovate once the pull request's checks pass. Minor and major
  updates only open a pull request.
- It relies on `platformAutomerge: false` from the default preset. A branch
  with no checks at all is never green to Renovate, so it is never merged:
  the repository should run its checks on every pull request, with no paths
  filter.
- It is not part of the default preset because a preset's `packageRules` win
  over a repository's top-level `"automerge": false`, so a repository that
  must not merge on its own could not turn it off.

## Overriding in a repository

A repository's own settings are applied after the presets it extends, and its
`packageRules` come after the presets' rules. A later matching rule wins, so a
repository can:

- set `"automerge": false` in a rule to keep some updates to a person,
- give some dependencies another `minimumReleaseAge`, or
- move some of the workflows' dependencies out of the `github-actions` group by
  giving them another `groupName` (for example a Node kept in step with the
  image's Node).

## Changing these presets

**A change here reaches every repository that extends them** on Renovate's
next run, as soon as it is on `main`. A preset Renovate cannot read makes it
open a configuration error issue in each of those repositories instead of
updating them. The `validate` workflow checks the files on every pull request,
but it does not see how a change combines with each repository's own rules, so
read those before changing or removing a rule here.
