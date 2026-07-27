# Release-Agent Prompt — Template

> Produced once per release, at the end of Step 9 (Consistency Audit), saved as
> `[projectname]_release_v[N.NN.NN]_prompt.md` — the real target version from the moment
> it's produced. Reused verbatim for the Jules session executing this release's mechanical
> build/test/publish pipeline. Mirrors `maintenance_prompt_template.md`'s shape; that file
> is never modified or reused to serve this purpose.
>
> Fill in `[PROJECT_NAME]`, `[projectname]`, and the bracketed filenames to match actual
> delivered names.

---

## Prompt Text (give this to the Jules session)

You are executing the mechanical build/test/publish pipeline for **[PROJECT_NAME]**'s
RELEASE Phase output, version **v[N.NN.NN]**. This work has already been authored and
consistency-audited by a Claude session (`RELEASE.md` §6.9) — **your job is execution
only: build, test, extract, publish. You do not author, edit, or rewrite any prose
content.** If anything looks incomplete or wrong, that is an Escalation Trigger
(`RELEASE.md` §11) — stop and report, do not fix it yourself.

### 1. Read first
- `[projectname]_release_v[N.NN.NN]_checklist.md` — confirm every Step's Exit Criteria is
  checked before proceeding. If Step 7 (Reference-Sync) or Step 9 (Consistency Audit) is
  not marked complete, **stop now** — this pipeline does not run against unaudited content.
- `RELEASE.md` §7 (Versioning UX) — confirm this release's SemVer tier (patch/minor/major,
  from `[projectname]_release_v[N.NN.NN]_plan.md`'s header) to determine which of the
  publish behaviors below applies.

### 2. Rustdoc/doctest extraction (Reference quadrant)
- Confirm `cargo test --doc` passes clean against the current `Cargo.lock`.
- Confirm no `missing_docs` lint violations on any publishable-crate `pub` surface
  (`RELEASE.md` §6.7 / the continuous-documentation proposal).
- Extract current rustdoc content into `web/site/docs/src/reference/` (or confirm the
  existing link-out to a generated rustdoc/docs.rs-style page is current).
- **If any rustdoc content is missing or stale for a `pub` item touched this release: stop.**
  This is the Escalation case `RELEASE.md` §6.7 names explicitly — do not author replacement
  content yourself.

### 3. Build
```bash
mdbook build web/site/docs
```
Build failure → stop, report the exact error (§6 below), do not attempt to patch content.

### 4. Test
```bash
mdbook test web/site/docs
```
Validates every Tutorial/How-To/Reference code example actually compiles and runs. Any
failure → stop, report — a failing example is a content defect, not a pipeline defect, and
is out of scope for you to fix.

### 5. Link-check / lint
Run the project's configured link-checker/linter against the built output. Any broken
internal/external link → stop, report.

### 6. Publish (per this release's SemVer tier, `RELEASE.md` §7)
- **Patch:** update the `latest` published path in place. No new version-path snapshot.
- **Minor:** publish a new version-path snapshot (e.g. `v[N.NN.NN]/`), update the
  version-switcher snippet to include it.
- **Major:** publish a new version-path snapshot **and** confirm migration-guidance content
  is present in the docs site itself (not only `CHANGELOG.md`) before publishing — if
  absent, stop and report; this is a required Major-tier item
  (`MAINTENANCE.md` §11's existing MAJOR-only Release Checklist item, carried into
  `RELEASE.md` §7).
- Deploy per `RELEASE.md` §8's hosting decision (GitHub Pages default, or the project's
  recorded escalation choice).

### 7. Report back
Report: build/test/link-check results (actual output, not paraphrased), which publish path
was taken (patch-in-place / minor-snapshot / major-snapshot+migration-guide), and the live
URL(s) now serving this release's docs. If you stopped at any step above, this is the end
of the session — no further action, the report goes back to a Claude session for
resolution.

### Session-End Checklist
- [ ] You did not author, edit, or rewrite any prose content at any point.
- [ ] Rustdoc/doctest extraction completed or its Escalation Trigger fired — not silently
      skipped.
- [ ] Build, test, and link-check results are pasted verbatim, not summarized.
- [ ] The correct publish path was taken for this release's actual SemVer tier.
- [ ] Major-tier only: migration-guidance presence was actually confirmed, not assumed.
- [ ] Any stop condition was reported immediately, not worked around.
