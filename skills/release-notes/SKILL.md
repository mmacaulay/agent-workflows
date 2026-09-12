---
name: release-notes
description: Turn everything merged since the last App Store release into human-readable release notes (a CHANGELOG.md entry and short App Store "What's New" copy), sourcing from merged PRs and closed issues rather than raw commit subjects, and checking in on anything ambiguous. Use before submitting a build. Invoke as /release-notes, or /release-notes v1.1.0 to scope from a specific tag.
---

Produce release notes a person would actually want to read. Two audiences, two outputs, one pass
over the same material.

`release-prep` calls this for its changelog step. Keep the generation logic here so there is one
source of truth.

## 1. Establish the range

Release notes cover everything since the last **App Store release**, not the last TestFlight
build. Those are different tags: `v<version>` marks a release, `build/<N>` marks a build.

```bash
git tag --list 'v*' --sort=-v:refname | head -3
```

Use the most recent `v*` tag as the base unless the user named one. If there are none, fall back
to the first commit and say so.

State the range and the commit count before going further, so the user can correct the scope
before you spend effort on the wrong window:

```bash
BASE=$(git tag --list 'v*' --sort=-v:refname | head -1)
git log --oneline "$BASE"..origin/main --no-merges | wc -l
```

## 2. Gather material, in order of how much intent it carries

Commit subjects are the thinnest source. Prefer, in this order:

1. **Merged PRs.** Title and body, including the acceptance criteria this repo requires. This is
   where the reasoning lives.
   ```bash
   BASE_DATE=$(git log -1 --format=%aI "$BASE")
   gh pr list --state merged --limit 100 --search "merged:>=${BASE_DATE%T*}" \
     --json number,title,body,mergedAt,labels
   ```
2. **Closed issues**, which describe the problem in user terms rather than the fix in code terms.
   ```bash
   gh issue list --state closed --limit 100 --search "closed:>=${BASE_DATE%T*}" \
     --json number,title,labels
   ```
3. **Commits**, to catch anything that landed without a PR.
   ```bash
   git log --oneline "$BASE"..origin/main --no-merges
   ```

If this release pulls in a new `@maplesmith/perch-core` version, check that repo too: a fix that
lives in core is still a user-visible fix here.

## 3. Classify by audience, not by commit prefix

`feat:` and `fix:` describe the change; they do not tell you whether a user can perceive it. Sort
every item into exactly one of:

- **User-facing** - someone using the app would notice. New capability, changed behaviour, a bug
  they could have hit, removed functionality.
- **Internal** - refactors, dependency bumps, CI, tests, docs, packaging. Real work, no
  perceptible difference.

A large refactor with no behavioural change is Internal, however much effort it took. A one-line
fix to a crash is User-facing. **Removed** functionality is user-facing and must never be quietly
dropped from the notes.

## 4. Check in on what you cannot decide

Ask the user, in one batch, about:

- Items you cannot confidently classify as user-facing or internal
- Anything whose user-visible effect is unclear from the PR
- Removals or migrations, to confirm how much prominence they should get
- Whether an infrastructure change that users could notice indirectly is worth mentioning

Do not ask about items that are obvious. A batch of three real questions is useful; a
questionnaire covering every commit is not.

## 5. Write the CHANGELOG.md entry

Create `CHANGELOG.md` at the repo root with a `# Changelog` header if absent. Prepend below the
header:

```
## [<version>] - <YYYY-MM-DD>

### Added
- ...

### Changed
- ...

### Fixed
- ...

### Removed
- ...

### Internal
- ...
```

Omit empty sections. Rules for the prose:

- Describe the effect on the user, not the implementation. "Weather no longer gets stuck on a
  loading spinner when the app is misconfigured" beats "add unconfigured status to
  useBackendSession".
- One line per item. Merge several PRs into one line when they were one piece of work.
- Reference the issue or PR number in parentheses where it helps someone go deeper.
- No em-dashes, per the repo's style. Use a regular dash.
- Keep Internal terse. It exists so the record is complete, not to be read.

Write it with the Write or Edit tool so it lands on disk.

## 6. Write the App Store "What's New" copy

Different job. The field allows 4000 characters; people read about three lines, in a small font,
while deciding whether to tap Update.

- **3 to 5 bullets, user-facing only.** Nothing from Internal reaches this.
- Lead with whatever most changes someone's experience.
- Plain language. No issue numbers, no package names, no internal vocabulary.
- Name a removal plainly if one happened. Users notice a missing feature and a silent removal
  reads as a bug.
- No em-dashes.

Present it as a copy-paste block, since it goes straight into App Store Connect.

## 7. Report

- The range used, and the tag it was based on
- Counts: user-facing vs internal
- The CHANGELOG entry as written, and its path
- The "What's New" block
- Anything deliberately left out, and why

## Notes

- Never invent a change that is not in the material. If the range looks empty, say so rather than
  padding it.
- If something shipped that users will dislike, say it plainly in the notes. Burying a removal
  costs more in support mail than it saves.
- Re-running this is safe. If an entry for the version already exists in `CHANGELOG.md`, offer to
  replace it rather than adding a duplicate.
