# nk-rewrite-coverage

A [Claude Code](https://code.claude.com) skill. After rewriting a long document — a spec, a research report, a handbook — list what the old version had that the new one no longer mentions, and account for every item with a three-state verdict before the rewrite is accepted.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## What it does

- `rewrite_coverage.py old new` lists old headings, bold terms and table cells missing from the new text, with structure hints for truncation and term dumps.
- A ledger with one verdict per item (`[renamed]` `[merged]` `[restored]` `[dropped]`) is the acceptance criterion; percentages are printed nowhere because they mislead in both directions.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## Install

Copy the folder into your skills directory (the skill is the repository root):

```bash
git clone https://github.com/NickkkLian/nk-rewrite-coverage ~/.claude/skills/nk-rewrite-coverage
```

or inside one project: `git clone … .claude/skills/nk-rewrite-coverage`.

As a plugin, through the marketplace in the index repository:

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-rewrite-coverage@nickkk-skills
```

To try it for one session without installing: `claude --plugin-dir ./nk-rewrite-coverage`.

## Verify

```bash
python3 scripts/rewrite_coverage.py --selftest
```

Standard library only, Python 3.9+. Before publishing, the guarded lines of each script were
mutated one at a time in a sandbox copy and the self-test was confirmed to go red on the named
assertion, without a traceback; the unmutated control stayed green.

## License

MIT. Read a script before letting it run in your environment.
