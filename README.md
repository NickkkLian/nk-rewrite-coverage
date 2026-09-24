# nk-rewrite-coverage

![nk-rewrite-coverage](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-rewrite-coverage.png)

An agent skill for [Claude Code](https://code.claude.com) and [OpenAI Codex](https://developers.openai.com/codex). After rewriting a long document — a spec, a research report, a handbook — list what the old version had that the new one no longer mentions, and account for every item with a three-state verdict before the rewrite is accepted.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

![nk-rewrite-coverage demo: before and after](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/nk-rewrite-coverage.gif)

## What it does

- `rewrite_coverage.py old new` lists old headings, bold terms and table cells missing from the new text, with structure hints for truncation and term dumps.
- Structure hints (headings, table rows, quote lines, characters, old → new) flag a truncated new file and a term dump that kept the words but lost the sections.
- A ledger with one verdict per item (`[renamed]` `[merged]` `[restored]` `[dropped]`) is the acceptance criterion; percentages are printed nowhere because they mislead in both directions.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## How it works

1. Keep the old version (git has it: `git show HEAD:path > /tmp/old.md`)
2. `python3 scripts/rewrite_coverage.py old.md new.md --out unaccounted.md` prints every heading, bold term and table cell…
3. Read the hints first
4. Give every unaccounted item one verdict
5. `rewrite_coverage.py old.md new.md --ledger ledger.md` exits 0 only when every item has a tag
6. Anything tagged `[restored]`

## Install

Pick one of four ways: three for Claude Code, one for OpenAI Codex. Skills load when a session starts, so open a **new** session after installing.

### 1 · Terminal, one command

```bash
git clone https://github.com/NickkkLian/nk-rewrite-coverage ~/.claude/skills/nk-rewrite-coverage
```

1. Run the command above (for one project only, clone into `.claude/skills/nk-rewrite-coverage` inside that project).
2. Start a new Claude Code session.
3. Check it loaded: type `/nk-rewrite-coverage` — it appears in the slash-command menu. Or just ask for the task; the skill triggers on its own.

### 2 · Claude Code in a terminal session (plugin)

The plugin route goes through the [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) marketplace. Add it once; after that each skill is one command.

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-rewrite-coverage@nickkk-skills
```

1. In a Claude Code session, run the first line (once per machine).
2. Run the second line.
3. Start a new session (or run `/reload-plugins`). The skill shows up as `nk-rewrite-coverage:nk-rewrite-coverage`.

Without opening a session, the same two steps work from a shell: `claude plugin marketplace add NickkkLian/nickkk-skills` then `claude plugin install nk-rewrite-coverage@nickkk-skills`.

### 3 · Claude desktop app (Code tab)

**Add the marketplace first — Discover only searches marketplaces you have already added.**

<img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/panel-route.gif" alt="Adding the marketplace and installing a skill in the desktop app" width="640">

<sub>Recorded on 2026-09-16, when the marketplace listed ten skills, all at version 0.1.0; it lists more now. The repository list in this recording shows the recorder's own repositories because a GitHub account is connected; yours will show yours. Type the full name as in step 4.</sub>

1. In the chat box, type `/plugin marketplace` and press Enter (or open **Settings → Customize → Plugins**). The **Plugins** panel opens.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step1-type-plugin-marketplace.png" alt="/plugin marketplace typed in the chat box" width="480">
2. Top right, open **Add ▾** and choose **Add marketplace**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step2-add-menu.png" alt="The Add menu with Add marketplace" width="480">
3. Choose **Add from a repository**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step3-add-from-repository.png" alt="Add marketplace dialog: Add from a repository" width="480">
4. In **URL**, type the full `NickkkLian/nickkk-skills`. At the bottom of the list choose the row **Use "NickkkLian/nickkk-skills"**, then press **Sync**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step4-url-then-sync.png" alt="URL filled in, Sync button" width="480">
5. You land on **Discover**, filtered to the new marketplace (**Filter · 1**). Find **Nk rewrite coverage** and press **Add**. Installed ones show **✓ Added**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step5-discover-add.png" alt="Discover list with Added and Add buttons" width="480">
6. Close the panel and start a new session.

To try it for one session without installing anything: `claude --plugin-dir ./nk-rewrite-coverage` from a clone.

### 4 · OpenAI Codex CLI

```bash
git clone https://github.com/NickkkLian/nk-rewrite-coverage.git ~/.agents/skills/nk-rewrite-coverage
```

1. Run the command above (for one project only, clone into `.agents/skills/nk-rewrite-coverage` inside that project).
2. Start a new Codex session.
3. Check it loaded, without spending a model call: `codex debug prompt-input | grep -o -- '- nk-rewrite-coverage[a-z0-9:-]*' | sort -u` prints `- nk-rewrite-coverage:nk-rewrite-coverage:`. Codex adds the `nk-rewrite-coverage:` prefix because this repository also carries a Claude Code plugin manifest. Ask for the task and the skill triggers on its own, or type `$` and pick it from the list.

## Compatibility

| Agent | Tested | What was checked |
|---|---|---|
| Claude Code (CLI 2.1.173, macOS) | yes | In a fresh project with an isolated Claude config, inside a macOS sandbox that blocked reading the tester's ~/.claude folder (settings, session history, memory), Desktop, Documents and Downloads, SSH keys and git identity, a plain request that never names the skill triggered it and it ran its bundled script. The route 2 plugin commands were also run from a shell with an isolated config: marketplace add, install, list. |
| OpenAI Codex CLI (0.154.0-alpha.6.2, gpt-5.6-sol, low reasoning, macOS) | yes | Copied into `~/.agents/skills` of a temporary home (the folder route 4 clones into), in a fresh project, without the user's Codex config. From a plain request that never names the skill, Codex read SKILL.md, ran `scripts/rewrite_coverage.py` on the old and new spec, restored the dropped section verbatim, recorded each decision in a ledger and ended with nothing unaccounted. |
| Cursor, Gemini CLI | no | Not tested. Their documentation says both read `~/.agents/skills`, the folder route 4 clones into; Gemini CLI asks before it activates a skill. |

In this skill's Codex run, every call into the skill folder's scripts/ used that folder's absolute path. Route 4 was checked for this repository: cloned from GitHub into a temporary home's `~/.agents/skills`, it was listed by the step 3 command. This skill's frontmatter uses only name, description, license and metadata.

## Verify

```bash
python3 scripts/rewrite_coverage.py --selftest
```

Standard library only, Python 3.9+. Before publishing, the guarded lines of each script were
mutated one at a time in a sandbox copy and the self-test was confirmed to go red on the named
assertion, without a traceback; the unmutated control stayed green.

## Limits

- Content moved to the wrong section (the words exist, the place is wrong).
- A change in any other cell of a table row: each row is compared by one cell, the first, or the second when the first column only counts the rows (1, 2, 3 … in order). Codes and ids in a first column (200, #123) are compared as they are.
- Quotations that are no longer verbatim — compare those against the source, not against the old draft.
- A new file that is complete and wrong.
- Truncation of the new file is hinted by the character ratio only; confirm the file ends where it should.

## License

MIT. Read a script before letting it run in your environment.
