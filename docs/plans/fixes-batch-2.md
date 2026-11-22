# Plan: fixes-batch-2

## What Sean asked for
"ready", meaning: start the next batch of small Quickstart fixes for upstream, the three left over from fixes-batch-1 because they change browser code.

## What we found
Checked on upstream `develop` `7eba0f24`. All browser tests (1,915) now run on this machine with the team's shared environment.
- **Divider characters break a run.**
  - On the Kometa page, a divider of `"` or `\` builds a command Quickstart can't read back, so the run fails with a server error page.
  - The same happens to a run queued behind the Plex maintenance window. That one fails inside a background task, so it never starts and the error is lost.
  - On the Kometa launch paths, only one place reads the command without catching the error: `modules/process_lifecycle.py:157`.
- **Kometa's `--timings` option is missing from the Kometa page.**
  - Kometa added it in July 2026.
  - Kometa builds from before July 2026 ignore options they don't know, so there the box just does nothing. *(Corrected after research: builds from before mid-October 2026 can crash on startup with it. See item 2 and the Record.)*
- **Capital `INDEX(...)` in a processing mode.**
  - The browser accepts `INDEX(A-F)`, but Quickstart's server and Kometa's own config schema need the word `index` in lowercase. The letters inside the brackets can be either case.
  - So the browser says it's fine, and saving or starting the run then fails.
  - This is the fix dropped from batch 1, now made on the correct side.

## What we'll change
Each item becomes its own small upstream PR to `develop`, with its tests.

1. **Divider characters** (`static/local-js/modules/kometa/_runCommand.js`, `modules/process_lifecycle.py`, tests).
   - **Browser:** the divider field will refuse `"` and `\` with a clear message naming them. Every other character it accepts today is still accepted.
   - **Server:** if a run command can't be read, the launcher will return a normal "invalid run command" error instead of crashing. That covers both the Start button and the queued run after Plex maintenance.
   - *Check:*
     - New browser tests: `"` and `\` are refused; `'`, `#`, `$` and `=` still work.
     - New Python tests: a command ending in `--divider """` gives an error result, not a crash, and starts nothing.
     - Both ways a run starts are tested. The Start button returns the error message to the page. A run queued behind Plex maintenance logs the failure, and the maintenance check keeps running.
     - The new tests fail on today's code and pass with the fix.
2. **`--timings` option** (`_cliFlags.js`, `_runCommand.js`, `_headerBadges.js`, `900-kometa.js`, `templates/900-kometa.html`, tests).
   - Add a "timings" checkbox next to the other run options. It works like the existing ones:
     - it is added to the command when ticked;
     - it is counted in the page header's number of enabled options;
     - it gets a label and help text next to the checkbox, in both the plain-name and command-name views.
   - The help text will say: Kometa builds from before July 2026 ignore it, and builds from before mid-October 2026 can crash on startup with it. Update Kometa before turning it on.
   - *Check:* the existing flag list and option-count tests are updated. New tests check that `--timings` is added to the command when ticked, left out when not, and that ticking or unticking it rebuilds the command.
3. **Capital `INDEX(...)`** (`static/local-js/modules/processingMode.js`, tests).
   - The browser check will need the word `index` in lowercase, matching the server and Kometa's schema. The letters inside the brackets can still be either case, as the server allows. `added(...)` stays case-insensitive, because all three already accept that.
   - The dropdowns already build `index(A-F)`, so only text typed by hand is affected.
   - Loading a saved `INDEX(...)` value into the dropdowns still shows it as `index(...)`, as it does today. *(The audit found the row's saved text stays uppercase until the row is edited. See the Record.)*
   - *Check:*
     - New browser tests: `INDEX(A-F)` and `Index(a)` typed by hand are refused.
     - `index(A-F)`, `index(a-f)` and `index(A-F#)` are still accepted, and saved ranges still load into the dropdowns.
     - New server test: the server also refuses `INDEX(A-F)`. This pins today's behaviour.
4. **All project checks**, using the team's shared environment:
   - `npm test`, `npm run build` and eslint;
   - the full Python suite with Kometa's default files in place;
   - ruff, black (line length 180) and the commit-time checks on every file, as the README asks.
   - Each fix is also checked on its own on top of `develop`, since each becomes a separate PR.
5. **Upstream PRs.** After your final approval, each fix goes on its own branch, cut from upstream `develop`, with a prefilled link for you to open it. Each fills in every section of Quickstart's PR template.

## Who's on it
- One Engineer for all three, one commit per item.
  - Items 1 and 2 both touch `_runCommand.js`, but in different places.
  - Each commit must apply on its own to `develop`.
- Then QA, the Security Officer and the Auditor review the work.
- Research is done (above).
- This job counts as security-sensitive, because items 1 and 2 touch how the run command is built and read. So it also gets a second security review. If that reviewer's service is still out of allowance, the report will say so.

## Risks
- **Item 1** refuses two characters Kometa itself would accept as dividers. The divider is only decoration in the log, so the loss is tiny. The other way is to quote those two characters specially, which is easier to break later.
- **Item 2**
  - The box does nothing on Kometa builds older than July 2026.
  - Kometa fixed a startup crash in its timing code in October 2026 (#3719). A build from between July and October might crash if the box is ticked. The help text says "recent Kometa".
  - A version check like the processing-mode one would avoid both, but it would be a much bigger change. Left out unless you want it.
- **Item 3:** anyone who typed `INDEX(...)` by hand now sees the error straight away, instead of when saving.

## Out of scope
- Checking the installed Kometa before offering `--timings`.
- The `--run-files` field, Serializd genre sources, `--no-source-integrity` and `--delete-collections-labels`.
- Testing on Windows: Windows command reading is checked only through Python's Windows mode, not on a real Windows machine.
- The ImageMaid launcher (`modules/process_lifecycle.py:223`) has the same unguarded read, but nothing known builds a broken ImageMaid command. Noted for later.

---

## Record (2026-11-22)

**Status:** built and checked, with no open problems. All three fixes are ready to go upstream. This branch holds the plan, the three fixes and this record. It is never merged. After approval, each fix is copied to its own branch without these notes.

**What shipped on this branch:** three commits, each meant to become its own upstream PR. Each one applies on its own to upstream `develop` (`7eba0f24`) and passes its tests there.
- `d59909f0` Reject divider characters that break the Kometa run command
  - Changed: `_runCommand.js`, `modules/process_lifecycle.py`, `quickstart.py`.
  - The Start button gets a 400 error with a clear message.
  - Tests: JS, `test_start_stop.py`, `test_more_paths.py`.
- `ffb6b811` Add the --timings option to the Kometa run options
  - Changed: the five option lists and the template.
  - The help text warns about builds from before July and before mid-October 2026.
  - Tests: JS.
- `912448eb` Require lowercase index in processing mode values
  - Changed: `processingMode.js`.
  - Tests: JS, plus one server case.

**Checks:**
- **Browser tests:** `npm test` passes: 1,934 tests.
- **Build:** `npm run build` and eslint are clean.
- **Python tests:** the full suite, with Kometa's default files in place, gives 2,780 passed, 16 skipped, 0 failed.
- **Style:** ruff and black (line length 180) are clean.
- **Commit-time checks:** every check passes on all files except the README spell check, which fails the same way on upstream `develop`.
- **New tests:** they fail on upstream's code without the fixes. That is 10 browser tests and 3 Python tests.
- **Edge cases:** only `"` and `\` change as dividers. `'`, `#`, `$`, `=`, `é` and `—` still work. `INDEX(A-F)`, `Index(a)` and `indeX(a-f)` are now refused in the browser. `index(A-F)`, `index(a-f)`, `index(A-F#)` and `added(...)` in any case still work.

**Review rounds:**
- Plan check: five corrections, all taken before approval. The most important: the dropdowns build `index(A-F)`, so only the word `index` may be case-checked.
- QA: passed.
- Security: clear. The run command is still started as a list of words without a shell. The new error text holds only the parser's fixed message, and it is shown as plain text.
- Second security review: skipped. Its service has used up its monthly allowance.
- Audit: approved, with two should-fix points.
  - **Fixed:** an unreadable command now returns 400 (bad request), not 500, matching how the endpoint treats a bad processing mode.
  - **Not changed:** see Decisions.

**Decisions:**
- **Restored `INDEX(...)` rows stay uppercase until edited.** A schedule row loaded with `INDEX(A-F)` shows dropdowns that would build `index(A-F)`. The row's saved text stays uppercase until the row is edited, so saving it unchanged is refused by the server.
  - This was already true before this change.
  - Such a value is rare, because the server never saves one.
  - Left for a later batch, to keep this PR narrow.
- **Divider:** the browser refuses `"` and `\` rather than quoting them specially. Quoting would keep two more characters working, but it is easier to break later.
- **`--timings`:** offered without checking the installed Kometa version. Older builds ignore it, and the help text warns about the crash in builds from before mid-October 2026.

## Upstream PRs (for Sean to open after final approval)
Each one is opened from its own branch on the fork, cut from upstream `develop`, into `Kometa-Team/Quickstart:develop`.
1. `fix-divider-quote-backslash`: "Reject divider characters that break the Kometa run command" (Bug Fix)
2. `add-timings-run-option`: "Add the --timings option to the Kometa run options" (Feature/Tweak)
3. `fix-processing-mode-index-case`: "Require lowercase index in processing mode values" (Bug Fix)
