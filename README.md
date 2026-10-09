# nk-rewrite-coverage

An agent skill for [Claude Code](https://code.claude.com) and [OpenAI Codex](https://developers.openai.com/codex). Check what a document rewrite lost.

**What you get.** One real run of nk-rewrite-coverage 0.2.2, copied from the terminal on 2026-10-09:

```text
$ python3 scripts/rewrite_coverage.py examples/english-old.md examples/english-new.md --html demo/report.html
distinct items · kept 1 of 5 numbers, 2 of 3 links, 1 of 2 code items, 0 of 2 quoted strings, 1 of 2 tentative names; lost: 3 numbers, 1 link, 1 code item, 1 quoted string, 1 tentative name, 1 bold term, 1 table cell; fewer: 1 number (3→1 occurrences), 1 quoted string (3→1 occurrences)
old: 11 items (headings, bold terms, table cells) · unaccounted in new: 2
structure  headings 7→7  table_rows 4→3  quote_lines 0→0  chars 2878→2485
  [      ] bold       reserve envelope
  [      ] table-cell quiet slot
✘ 2 unaccounted item(s) without a verdict — decide each: [renamed], [merged], [restored], [dropped]
  lost  name          Mara Fen (1→0; old 5:274)
  lost  number        1,250 (1→0; old 9:27)
  lost  number        73 (1→0; old 9:51)
  lost  number        19% (1→0; old 9:102)
  lost  link          https://example.org/harbor/plan (1→0; old 9:515)
  lost  code          harborctl check --batch amber (1→0; old 13:97)
  fewer number        8 (3→1; old 13:230)
  fewer quoted-string carry the amber card (3→1; old 17:43)
  lost  quoted-string leave the side gate open (1→0; old 17:189)
Report: demo/report.html
```

**Measured, not guaranteed.** Three blind trials used six invented document pairs, with the losses listed by hand before each run.
On the two pairs this version had never seen, 35 items were really lost or appeared fewer times: it reported 34 of them, and 3 that were not lost.
Three of those four errors were names, the least certain kind.

![nk-rewrite-coverage](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-rewrite-coverage.png)

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## Try it

Nothing is installed and nothing under `~/.claude` changes: clone, run the self-test, run the example (it only writes inside the clone).

```bash
git clone https://github.com/NickkkLian/nk-rewrite-coverage && cd nk-rewrite-coverage
python3 scripts/rewrite_coverage.py --selftest
mkdir -p demo
python3 scripts/rewrite_coverage.py examples/english-old.md examples/english-new.md --html demo/report.html
```

The self-test prints:

```text
rewrite_coverage selftest · 161/161 passed
```

The last command prints the block at the top of this page; its last line is the one below, and its exit code is 1 (non-zero on purpose: it found something).

```text
Report: demo/report.html
```

![nk-rewrite-coverage demo: before and after](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/nk-rewrite-coverage.gif)

The demo above is a rendering of an earlier run and cuts its longest lines short; the block at the top of this page is a full run of this version.

## What it does

- `rewrite_coverage.py old new` takes every number written in digits (and number words up to twenty, except one and 一), every link, piece of Markdown code and quoted string in the old text, and reports each as kept, fewer (it now appears fewer times) or lost, in English and Chinese. Capitalised Latin names come last as tentative names, the least certain kind.
- One offline HTML page shows the old text with every lost item marked where it stood and fewer items in amber, a list that links to each place, and the new text beside it. Light and dark; it reads at phone width.
- Old headings, bold terms and one cell per table row that are missing from the new text are still listed; a ledger of verdicts (`[renamed]` `[merged]` `[restored]` `[dropped]`) annotates them.
- The exit code is 1 when anything is lost or fewer and 0 when nothing is; `--json` gives every item with its place.
- Matching is literal. It does not judge rephrased sentences, and what it misses on purpose is listed under Limits.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## How it works

1. Keep the old text in a separate file.
2. Run `python3 scripts/rewrite_coverage.py old.md new.md --html rewrite-coverage.html`.
3. Read the first line, then open the local report.
4. Review every lost or fewer item against the source and restore it or record why it changed.
5. Rerun after restoring details.
6. When you report to the user, give the tool's first line first, exactly as printed.

## Why it is built this way

**The idea.** A rewrite is a retelling from memory, and memory keeps conclusions and drops process. A 630-line analysis rewritten after a context compaction came back as 755 lines that read fine — and had lost eight reasoning chains compressed into one, a timetable, five verbatim quotations, a placement table and a cross-reference section, six whole blocks.

**Where it came from.** Own practice, 2026-08: the compaction rewrite described above, followed by four rounds of an auditor breaking the first version of this tool (a positive probe that was always true; a percentage that a term dump could game; a truncated file that still scored mid-range).

**Evidence.** What was broken on purpose to show that the self-tests can fail is under [Verify](#verify); what was run end to end, and in which agent, is under [Compatibility](#compatibility).

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
| Claude Code (CLI 2.1.173, macOS) | yes | In a fresh project with an isolated Claude config, inside a macOS sandbox that blocked reading the tester's ~/.claude folder (settings, session history, memory), Desktop, Documents and Downloads, SSH keys and git identity, a plain request that never names the skill triggered it and it ran its bundled script. The route 2 plugin commands were also run from a shell with an isolated config: marketplace add, install, list. This session was run on 2026-09-16 with version 0.1.x. On 2026-10-09 one session on 0.2.2 (model claude-opus-4-8), asked in plain words what a tidied runbook was missing, called the skill, ran the script once with `--html` and named all 17 lost or fewer items correctly; it stated no counts, did not say that names are the least certain kind, and put two findings of its own reading in the same list. |
| OpenAI Codex CLI (0.154.0-alpha.6.2, gpt-5.6-sol, low reasoning, macOS) | yes | On 2026-10-09 one session on 0.2.2 (Codex CLI 0.162.0-alpha.2, gpt-5.6-sol passed at low effort), with the same request, read SKILL.md and the matching rules, ran the script twice (with `--json` and `--html`) and named all 17 lost or fewer items correctly; it stated no counts, did not say that names are the least certain kind, and put two findings of its own reading in the same list. The rest of this row is a run of version 0.1.x, made on 2026-09-16. Copied into `~/.agents/skills` of a temporary home (the folder route 4 clones into), in a fresh project, without the user's Codex config. From a plain request that never names the skill, Codex read SKILL.md, ran `scripts/rewrite_coverage.py` on the old and new spec, restored the dropped section verbatim, recorded each decision in a ledger and ended with nothing unaccounted. |
| Cursor, Gemini CLI | no | Not tested. Their documentation says both read `~/.agents/skills`, the folder route 4 clones into; Gemini CLI asks before it activates a skill. |

On 2026-10-09 one plain-request session per agent was run on 0.2.2, one attempt each (the rows above); in both, the report page the session wrote is byte-identical to a direct run of the script on the same two files. Also run on 0.2.2, without an agent: the self-test (161 cases) on Python 3.13 and 3.9, the bundled examples, and the six document pairs of the measurement at the top of this page.

In this skill's Codex run, every call into the skill folder's scripts/ used that folder's absolute path. Route 4 was checked for this repository: cloned from GitHub into a temporary home's `~/.agents/skills`, it was listed by the step 3 command. This skill's frontmatter uses only name, description, license and metadata.

## Verify

```bash
python3 scripts/rewrite_coverage.py --selftest
```

Standard library only, Python 3.9+. On 2026-10-09 every self-test above passed, and
`breakcheck.py` from [nk-breakable-selftest](https://github.com/NickkkLian/nk-breakable-selftest) broke each script on purpose in a sandbox copy:

- `rewrite_coverage.py`: 62 lines broken one at a time; 21 turned the self-test red without a traceback. Not covered: the self-test stayed green with L323, L382, L383, L396, L397, L402, L676, L1400, L1439, L1448, L1455, L1461, L1466, L1473, L1483, L1488, L1489, L1504, L1518, L1532, L1541, L1546, L1554, L1560, L1571, L1572, L1578, L1579, L1601, L1605, L1609, L1613, L1621, L1645, L1647, L1657, L1660, L1662, L1870, L1885 switched off; switching off L121 crashed the script instead of failing a sample, which does not count as caught.

The unmutated control stayed green every time. Only lines that record a finding, raise, or return a failing exit code
were broken (the tool's pattern, or the hand-written list); a line number refers to the script as shipped in this version.
This shows those lines are covered. It does not show that nothing else can fail.

## Limits

- The tool does not judge rephrased sentences, lost reasoning, facts moved to the wrong section, or digits whose meaning changed.
- Names are last as tentative names, the least certain kind.
- It deliberately misses one-word names only at sentence/heading starts, names that are months, and single label values such as Friday in Next meeting: Friday.
- Lowercase names, Chinese personal names, titles, units, currencies and CSV are excluded.
- Interior capitalisation can remain ambiguous.
- Number words above twenty and standalone one/一 are deliberately ignored, even for genuine quantities.
- A table row is compared by its first cell, or its second when the first column counts rows.
- Other cells can change without a structural warning.
- Code and links use the documented text/Markdown forms, not a full Markdown parser.
- An opening word glued to a name can be reported as a lost name: `On Windows` was reported lost while Windows was still in the new text.
- The first word of a table cell can be taken for a name.
- A small number can be hidden when the same digit appears in the new text as a label, such as "Step 3", a numbered table row or a footnote mark: the lost number is then counted as kept.

## License

MIT. Read a script before letting it run in your environment.
