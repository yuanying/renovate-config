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
and `github>yuanying/renovate-config:platform-automerge` are opt-in, and a
repository extends at most one of them; leave both out where every merge is
left to a person.

## Presets

### `default.json`

- Extends `config:recommended` and `:semanticCommitTypeAll(chore)`: in a
  repository whose history Renovate reads as using semantic commits, every
  update is a `chore(deps):` commit. Elsewhere the commits keep Renovate's
  plain `Update ...` subjects.
- `timezone` is `Asia/Tokyo` and the Dependency Dashboard is on.
- Updates are proposed once a week: `schedule` is `* * * * 0`, the whole of
  Sunday in Tokyo. The repositories release what Renovate merged as a patch
  version on Monday at 09:00 in Tokyo, so what opens on Sunday can make that
  release, and a self-hosted Renovate that runs once a day still meets a
  window of a whole day. The schedule only holds back new branches: a pull
  request already open is still rebased and moved to a newer release during
  the week, and what a repository lets Renovate merge is still merged then.
  A box ticked on the Dependency Dashboard also opens its branch at once.
- A fix for a known vulnerability does not wait for Sunday. Renovate ignores
  `schedule`, `minimumReleaseAge` and the hourly limits for
  `vulnerabilityAlerts`, which come from the repository's Dependabot alerts
  and so need them turned on.
- `osvVulnerabilityAlerts` is on: Renovate also opens these fixes from the
  OSV database, which it downloads whole and queries locally. It needs no
  repository setting and sends nothing about the repository, so it works for
  the Renovate app, a self-hosted Renovate and private repositories alike. It
  covers the packages of languages (npm, PyPI, Go modules, RubyGems, crates
  and the like), and only direct dependencies; it has nothing for Docker
  images, Kustomize or GitHub Actions.
- `minimumReleaseAge` is `2 days`, with `internalChecksFilter: strict`: an
  update gets no branch until it has been public for two days.
- `platformAutomerge` is `false`: whatever Renovate is allowed to merge, it
  merges itself once it sees the checks green, without the repository's
  auto-merge setting or branch protection.
- The two warnings Renovate gives when it lets a release without a release
  time through under `minimumReleaseAgeBehaviour: timestamp-optional` are
  logged as `info`, so they stay out of the Dependency Dashboard's repository
  problems. That is the behaviour the repository asked for; every other
  warning is left as it is.
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

### `platform-automerge.json`

- Every update, major ones included, gets GitHub's auto-merge
  (`platformAutomerge: true`, `automerge: true`) as soon as its pull request
  is opened. GitHub merges it the moment the default branch's ruleset is met,
  without waiting for Renovate's next run.
- What holds an update back is the repository's ruleset, not Renovate. Before
  a repository extends this preset it must:
  - allow auto-merge,
  - list its reviewers in `.github/CODEOWNERS`, and
  - protect the default branch with a ruleset that requires a pull request
    with a code owner's approval, dismisses an approval when new commits are
    pushed, and requires the checks of every pull request.
- Without those, Renovate cannot turn on auto-merge and falls back to merging
  every update itself once the checks are green, major updates included.
- `automerge` is set at the top level, so a repository's own top-level
  `"automerge": false` still turns it off.

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

The workflow runs a pinned Renovate rather than the latest release, so that an
option upstream starts asking to migrate does not fail every pull request at
once. Renovate updates the pin in a pull request of its own, outside the
`github-actions` group: a patch is merged once the presets still validate, and
a release that asks for a migration fails only that pull request. Making the
migration is a separate decision: the repositories read these presets with
whichever Renovate runs on them, which may not know the new form yet.
