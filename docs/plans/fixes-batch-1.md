# Plan: fixes-batch-1

## What Sean asked for
"review https://github.com/Kometa-Team/Quickstart for work similar to what another team does for custom-components/pyscript", then: "I recreated the fork".

## What we found
The review (2026-11-22, upstream `develop` at `7eba0f24`) found:
- **Quickstart has no open bug reports.** The maintainer fixes reported bugs within days. The six open issues are either already built (#1958, #1914) or large redesigns the maintainer owns (#1844, #1602, #1335, #1332).
- Quickstart has no rule against outside contributions like ours, and small outside PRs to `develop` get merged within hours. PRs are squash-merged, and the PR template asks for linked issues, the PR type, a description with testing, and the test environment.
- **Five small problems nobody has reported yet.** This batch takes the three that can be fully tested on this machine. The other two change browser code, and the browser-code tests crash on this machine.

## What we'll change
Each item becomes its own small upstream PR to `develop`, with its tests.

1. **README: two outdated statements** (`README.md` only).
   - The `docker run` example has no `-p 7171:7171`. If you copy it, Quickstart starts but your browser can't reach it. The `docker compose` example below it already publishes the port.
   - The Logscan "Scope" line still lists "an opt-in automatic-upload setting" as a follow-up. The "Automatic support uploads" paragraph above it describes that setting as finished.
   - *Check:* the corrected command matches the compose example and the app's default port (7171). The spelling check (pyspelling, which uses `.github/.wordlist.txt`) passes on `README.md`.
2. **The `mal_all` genre source is lost on import and export** (`static/json/quickstart_attributes.json`, `modules/dependency_reasons.py`, tests).
   - Kometa accepts `mal_all` (MyAnimeList genres, including explicit genres, themes and demographics). Quickstart's list of genre sources only has `mal`. So an imported `[mal_all, tmdb]` is stored as the source `tmdb` plus a made-up genre called "mal_all", and exported as `[tmdb, [mal_all]]`. That changes both the meaning and the order.
   - The fix adds `mal_all` to the list, next to `mal`. It also marks `mal_all` as needing MyAnimeList sign-in, the same way `mal` does.
   - *Check:* a new test imports `[mal_all, tmdb]` and exports it unchanged, in the same order. Another test checks that MyAnimeList sign-in is reported as needed. Real made-up genres still export as before. The new tests fail on today's code and pass with the fix.
3. **The server rejects capital letters in `index(...)` processing modes** (`modules/library_scheduling.py`, `tests/test_library_scheduling.py`).
   - The browser accepts `INDEX(A-F)` (its check ignores case), and so does Kometa, which lowercases modes. Quickstart's server check and its saved-settings schema only accept lowercase `index`. So the browser accepts a value that the server then refuses.
   - The fix makes the server check and the schema ignore case for `index(...)`, matching the browser. Nothing else changes.
   - *Check:* new test cases accept `INDEX(A-F)` and `Index(a-f#)`. The existing rejections, such as a backwards range like `index(F-A)`, still fail. The new tests fail on today's code and pass with the fix.
4. **All project checks** on Python 3.13, as CI runs them: `pytest` (`-m "not e2e"`, with both `requirements.txt` and `requirements-dev.txt` installed), `ruff check`, `black -l 180 --check` on changed files, eslint, and pyspelling on Markdown.
5. **Upstream PRs.** After your final approval, each fix goes on its own clean branch, cut from upstream `develop`, on your fork. You get a prefilled link for each one, to open the PR from your account. Each PR fills in every section of Quickstart's PR template.

## Who's on it
- One Engineer for all three items. They touch separate files.
- Then QA, the Security Officer and the Auditor review the work.
- Research is done (the review above). There's no second security review, because no security-related code is touched.

## Risks
- **Item 3:** the maintainer added processing modes last week, so he may change this code again. The fix is kept small to keep any clash simple.
- **Item 2:** if Quickstart's genre list is also used to build a menu, `mal_all` will appear there as a new choice. That is expected, and its label will copy Kometa's wording.
- **Overlap:** there are no open upstream PRs on these files, only dependency updates.

## Out of scope
- **Divider characters `"` and `\` breaking the run command, and the missing `--timings` option.** Both change browser code whose tests can't run on this machine. They are for a later batch, once there is a way to run those tests.
- **Making `FULL` and `DIFF` acceptable in capitals.** That would change the browser check too.
- **The three Serializd genre sources with the same problem as `mal_all`.** They apply only to TV shows and need their own handling.
- **New service pages, the `--run-files` field, and the other missing Kometa options.**
- **Notes:** the fork stays identical to upstream. This plan and its record live only on the `team/fixes-batch-1` branch, which is never merged.
