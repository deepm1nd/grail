# Agent Instructions

## 1. Canonical Source of Instructions
**MANDATE:** The instructions for operating within this repository are not contained
directly within this file. The single, canonical source of truth for all agent
instructions is the `main` branch of the `deepm1nd/grail` repository — this file is a
forwarding stub only.

## 2. Action Required
**MANDATE:** Before taking any other action, fetch the **latest commit on `main`** of the
following file — never a cached, previously-fetched, or otherwise remembered copy from
earlier in this session, even if grail content already appears elsewhere in context:

`https://github.com/deepm1nd/grail/blob/main/AGENTS.md`

Then fetch whichever files within that repository's `agents/` directory `AGENTS.md` (and
the phase file for your current Design/Development/Maintenance/Release work) direct you
to read for your current task — consult that phase file's own `§0` reading table
(`agents/DESIGN.md`, `agents/MAINTENANCE.md`, or `agents/RELEASE.md`) to fetch only the
files your current Step/M-stage/Step actually needs, not the full grail set by default.

Follow those instructions for the remainder of this session. This stub file itself is
ignored once the redirection above has been carried out.

## 3. If the Fetch Fails
If `main` cannot be reached (network restriction, repository access failure, or any other
fetch error): **stop and tell the user directly** — do not proceed on stale, cached, or
remembered instructions, and do not silently guess at grail's current rules. State plainly
that the fetch failed and ask the user to paste the needed grail files into the
conversation instead. If you already know which phase/Step you're working, name the
specific files that phase's reading table would call for, so the user can paste only
those rather than the entire grail file set.

## 4. Session-Start and Re-Check Discipline
At the start of the session, once `AGENTS.md` is fetched, note the commit/version it
reports (`CHANGELOG.md`'s latest entry) so any later re-check has something to compare
against. **Do not re-fetch continuously.** Re-check only at natural checkpoints — a
Design Step boundary, a Maintenance M-stage/Task-Group boundary, a Release Step boundary —
not mid-Step. A rule change discovered between checkpoints applies starting at the next
checkpoint; it does not retroactively reopen work already in progress under the version
that was current when that work began.
