# test-expert

A Claude Code subagent that reviews automated tests and tells you which ones you can't trust.

## Why

Coding agents are bad at validating their own work. They write tests that sound great on the surface but are often flaky, slow, or don't turn red when they should. A green suite built that way gives you false confidence: the build passes and the bug ships anyway.

test-expert goes deep on a change or a whole repo to surface those problems and help you prioritise them by impact. It hunts for:

- **Tests that lie:** assertions that can't fail, tautologies, expected values pasted from the code's own output, loops that assert nothing when the result is empty, placeholder-passable checks.
- **Coupling to structure:** mocks of your own code, tests reaching private internals, tests that break on a pure refactor.
- **Flakiness and slowness:** sleeps, wall-clock and randomness dependence, order-dependent tests, integration tests hiding in the unit suite.
- **Missing cases:** boundaries, degenerate inputs that can't tell the right operator from a wrong one, failure paths with no inverse test.
- **Wrong level:** E2E tests doing a unit test's job, and what to shift left.

Every finding is filed P0 (tests that lie) to P3 (cleanup) with a concrete action, plus architecture notes for problems the tests can't fix on their own. It also covers lint rules that catch these smells mechanically, and mutation testing to prove the suite fails when the code is broken.

## Install

**As a plugin** (Claude Code 2.1.275 or later), in Claude Code:

```
/plugin install test-expert --marketplace abrainwood/test-expert-agent
```

Plugin updates arrive through `/plugin`. The agent's name gets the plugin prefix, `test-expert:test-expert`, so use that name if you refer to it from your own agents or skills.

**Or copy the file** into your Claude Code agents directory. You manage updates yourself, and the agent keeps the plain name `test-expert`:

```bash
# for all your projects
mkdir -p ~/.claude/agents && cp agents/test-expert.md ~/.claude/agents/

# or for one repo
mkdir -p .claude/agents && cp agents/test-expert.md .claude/agents/
```

It defaults to Opus at medium effort. Change `model` and `effort` in the frontmatter to suit.

For architecture findings, the agent hands off using the vocabulary of two skills from [Matt Pocock's skills](https://github.com/mattpocock/skills): `/codebase-design` and `/improve-codebase-architecture`. They're optional. Without them the agent still files the same findings; you just pick up the architecture notes yourself.

## Use

Ask Claude Code for a test review and it routes to the agent:

- "Run a test review of the codebase"
- "Have test-expert review the tests on this branch against origin/main"
- "Review the tests I just wrote"

For a change review, give it the base ref so it can tell what the change introduced from what was already there.

## Does it work?

It was evaluated against [ledgerly](https://github.com/abrainwood/ref-app-for-test-review-agent-eval), a small public Python invoicing app (a calculator CLI, invoice pricing with volume discounts and progressive tax, a monthly customer report, an exchange-rates client) seeded with 44 known test defects and 10 decoys (tests that look suspicious but are fine). Point the agent at it to see what a review looks like. The agent's own examples are set in the same invoicing world but are deliberately invented, not the planted defects, so the eval still measures what the agent notices rather than what it was told.

On Opus 5.5 at medium effort, reviewing the whole suite:

| | Defects found | Cost per review |
|---|---|---|
| test-expert | 77-84% | ~$0.90 |
| Plain Claude Code, asked to "run a test review of the codebase" | 0-32% (avg 9%) | ~$0.28 |

Neither flagged a decoy. In 3 of 4 runs, plain Claude reviewed the latest commit instead of the tests. Small sample (2 and 4 runs), one app, one language: treat it as a signal, not a benchmark.

## License

MIT. See [LICENSE](LICENSE).
