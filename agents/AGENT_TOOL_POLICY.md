# Agent Tool Policy

**MANDATE: Adherence to this guide is not optional. It is a strict requirement.**

Governs the agent's own platform/execution tools (filesystem, submit/push, repo-history,
PR-comment tools) — distinct from `agents/SCRIPT_RULES.md` (script/command execution
content) and from `agents/PREFERRED_TOOLS.md`/`agents/PREFERRED_DEPENDENCIES.md` (Rust
dev-tool and library choices). Referenced from `AGENTS.md` §2.2. Supersedes the tool-tiering
content of a prior, now-repurposed file of the same conceptual role; tool names below match
the platform's current vocabulary.

See `CHANGELOG.md` for version history.

---

## 0. Relationship to Other Mandates

- **`submit` is the only push primitive** — there is no separate plain "commit without
  push." Every checkpoint discussed in `AGENTS.md` §2.1 (task-level submit cadence, WIP
  checkpoints) and `agents/DEVELOPMENT.md` §5.1/§5.2 is a `submit` call under this policy.
- **Code Review Policy** (whether `request_code_review` is called at all) is governed by
  `AGENTS.md` §2.2, not here — this file governs only which tier `request_code_review`
  itself sits in when it *is* used.
- **`reset_all()` and `restore_file()`** are already forbidden outright by `AGENTS.md` §2.1's
  additive-only mandate; this file's Forbidden-tier confirmation ritual is the *mechanism*
  for the rare, explicitly-approved exception — it does not loosen §2.1's default.

---

## 1. Allowed Tools

Usable at any time without special approval — read-only, planning, or communication tools
with no destructive or state-committing effect:

- `list_files`, `read_file`, `view_text_website`, `view_image`, `read_image_file`,
  `read_media_file`
- `set_plan`, `request_plan_review`, `plan_step_complete`, `record_user_approval_for_plan`
- `message_user`, `request_user_input`
- `google_search`
- `initiate_memory_recording`
- `run_in_bash_session` — subject to `agents/SCRIPT_RULES.md` for command content; the tool
  call itself requires no separate approval
- `pre_commit_instructions`, `frontend_verification_instructions`,
  `frontend_verification_complete`, `start_live_preview_instructions`

**Selective Reading Mandate and No Broad Repository Scan Mandate (`AGENTS.md` §2.2) still
apply** to `list_files`/`read_file` even though they're Allowed-tier here — being
approval-free is not license to whole-file-read a large document or enumerate the repo
speculatively.

---

## 2. Tools Requiring Approval

Propose the action and wait for explicit consent before calling:

- `write_file` (new file), `replace_with_git_merge_diff` (targeted edit to an existing file)
- `rename_file`
- `submit` — **propose the exact submit type (task-complete / Code-only / WIP checkpoint),
  its title, and whether code review is invoked (per `AGENTS.md` §2.2's current setting)
  before calling.** Per `agents/DEVELOPMENT.md` §5.2, the Development Plan pre-declares
  Submit Points per task at drafting time — this approval step confirms execution of an
  already-planned checkpoint, not a fresh judgment call each time.

**After every `submit`, the session stops and awaits the user's next message before
continuing any further work — per `agents/DEVELOPMENT.md` §5.2, the user says "Continue" or
"Proceed" (plain language, not the `AGENTS.md` §3.1 `APPROVED` token, which governs a
different mechanism) to resume. This pause is normal flow, not an error and not itself an
Escalation Trigger.**

---

## 3. Forbidden Tools

**MANDATE:** To use any tool below, the agent MUST follow the two-step confirmation
process: (1) propose the action, explicitly stating it is a Forbidden-tier tool and why it
seems necessary; wait for approval; (2) issue a second, distinct request beginning with the
exact phrase **"ARE YOU SURE?"**; only proceed on explicit confirmation to that second
request.

- `request_code_review` — see `AGENTS.md` §2.2's Code Review Policy; while bypassed, this
  tool is never called at all, Forbidden-tier or not.
- `delete_file`
- `reset_all` — additionally forbidden outright by `AGENTS.md` §2.1; the two-step ritual
  here governs only the rare case the user explicitly initiates this themselves.
- `restore_file` — same relationship to `AGENTS.md` §2.1 as `reset_all` above.
- `read_pr_comments`, `reply_to_pr_comments`

---

## Appendix

See `CHANGELOG.md` for this file's full version history.
