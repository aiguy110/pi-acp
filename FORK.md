# Tandem fork of pi-acp

This fork exists so Tandem can show a live context-usage meter for `pi` sessions.
It is tracked by Tandem's ACP fork-tracking system (`tandem acp` — see the
`acp-server-forks` skill in the Tandem home-base repo).

- **Upstream:** https://github.com/svkozak/pi-acp
- **Fork branch:** `tandem` — upstream `main` plus the commits below.
- **`main`** is kept as a plain mirror of upstream so `git merge --ff-only
  upstream/main` always works; never commit to it.

## Carried commits

1. **`feat(acp): report context window usage as usage_update`** — the actual
   feature. Written to be offerable upstream as-is. Upstream already has five
   open/closed-unmerged PRs for this same gap (#75, #87, #97, #114, #119), so
   Tandem carries its own until one of them lands.

2. **`chore: build on prepare so git installs produce dist/`** — packaging only,
   not for upstream. npm runs `prepare` (not `prepack`) when installing a git
   dependency; without it `npm install <git-url>#<ref>` yields a package with no
   `dist/`, and Tandem installs this fork straight from a pinned git SHA.

## Rebasing onto a new upstream release

```bash
git fetch upstream
git checkout main && git merge --ff-only upstream/main
git checkout tandem && git rebase main
npm install && npm run typecheck && npm run lint && npm test
```

Then push and record the new baseline so Tandem stops flagging the release:

```bash
git push --force-with-lease origin tandem
tandem acp fork pi --rebased-onto <new-upstream-version>
```

## If the feature is upstreamed

Check whether upstream now emits `usage_update` (`git log upstream/main --grep
usage_update`, or look for `sessionUpdate: 'usage_update'` in
`upstream/main:src/acp/session.ts`). If it does, retire the fork instead of
rebasing it:

```bash
tandem acp upstream pi --constraint '^<version-that-has-it>'
```

That returns Tandem to the published npm package and drops the fork tracking.
