# Tandem fork of pi-acp

This fork exists so Tandem can show a live context-usage meter for `pi` sessions
and deliver streamed output without building up a token-sized ACP notification
backlog.
It is tracked by Tandem's ACP fork-tracking system (`tandem acp` — see the
`acp-server-forks` skill in the Tandem home-base repo).

- **Upstream:** https://github.com/svkozak/pi-acp
- **Fork branch:** `tandem` — upstream `main` plus the commits below.
- **`main`** is kept as a plain mirror of upstream so `git merge --ff-only
  upstream/main` always works; never commit to it.

## Carried commits

Upstream 0.0.34 added context-usage reporting (#97): `usage_update` on session
new/load, model change, and before `agent_settled` resolves a prompt. The fork's
original usage commit was dropped in favour of it.

1. **`chore: build on prepare so git installs produce dist/`** — packaging only,
   not for upstream. npm runs `prepare` (not `prepack`) when installing a git
   dependency; without it `npm install <git-url>#<ref>` yields a package with no
   `dist/`, and Tandem installs this fork straight from a pinned git SHA.

2. **`fix: settle prompts after exhausted retries`** — resolves a prompt when
   Pi exhausts automatic retries without emitting a final `agent_settled` event.

3. **`fix(acp): adaptively coalesce streaming deltas`** (+ **`tune(acp): use
   100ms delta coalescing window`**) — batches adjacent text and thought deltas
   before ACP delivery. The batching window grows when the notification queue
   or observed delivery latency rises, preventing generation from outrunning
   Tandem's consumer while keeping low-pressure latency small. Offered upstream
   as #134.

4. **`feat(acp): publish context usage on assistant message_end`** — upstream
   only reports usage at turn boundaries; this keeps Tandem's meter live
   through long multi-step turns.

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

Context usage is already upstream. The fork can be retired once upstream also
settles prompts after exhausted retries (`auto_retry_end` with
`success: false`), coalesces streaming deltas (#134), and publishes usage
mid-turn (or Tandem accepts turn-boundary-only updates). Then:

```bash
tandem acp upstream pi --constraint '^<version-that-has-it>'
```

That returns Tandem to the published npm package and drops the fork tracking.
