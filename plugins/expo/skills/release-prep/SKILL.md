---
name: release-prep
description: Prepare for a new release - bump version, verify environment config is abstracted, generate changelog from recent commits, and run final checks. Use before building for TestFlight or App Store submission.
---

Prepare the project for a release. Walk through each step and report status.

## 1. Determine version bump

- Read current version from `app.json` (both `version` and `ios.buildNumber`)
- Check recent git history to determine if this is a major, minor, or patch bump:
  ```bash
  git log --oneline -20
  ```
- Ask the user to confirm the new version number before proceeding
- Suggest semantic versioning: breaking changes = major, new features = minor, fixes = patch

## 2. Bump version

- Run `npm run bump-build` if the project has it. It atomically increments `ios.buildNumber` in
  both `app.json` and the iOS `Info.plist`. Never edit those files by hand: they must stay in
  sync or Xcode archives with the wrong build number.
- The `Info.plist` path is derived from the app name rather than hardcoded, so this works in any
  Expo project:
  ```bash
  PLIST="ios/$(node -p "require('./app.json').expo.name")/Info.plist"
  ```
- If the version string (`version` in `app.json` and `CFBundleShortVersionString` in `Info.plist`) also needs a bump, update both files manually and confirm they match.
- If `android.versionCode` exists, increment it too.
- Commit the bumped files before proceeding:
  ```bash
  git add app.json "$PLIST" && git commit -m "chore: bump build to <N>"
  ```

## 3. Create and push a build tag

- Create a git tag named `build/<buildNumber>` (e.g. `build/8`) at HEAD - after the bump commit above
- Push the tag to origin immediately so it's available for future changelog scoping:
  ```bash
  git tag build/<N> && git push origin build/<N>
  ```
- When shipping to the App Store (not just TestFlight), also create and push a `v<version>` tag (e.g. `v1.0.0`) on the same commit.

## 4. Release notes and changelog

Delegate to the `release-notes` playbook from the workflows plugin rather than repeating its
logic here, so there is one source of truth for how notes get written.

It scopes from the last `v*` tag, sources from merged PRs and closed issues rather than raw
commit subjects, checks in on anything it cannot classify, then writes the `CHANGELOG.md` entry
and the App Store "What's New" copy.

If the user wants to skip it, the minimum is still a `CHANGELOG.md` entry for this version
describing user-visible changes, plus a note that the App Store copy is outstanding.

## 5. Environment & config check

- Verify no secrets or API keys are committed to source (check git-tracked files only):
  ```bash
  git ls-files | xargs grep -l 'EXPO_PUBLIC_' --include='*.ts' --include='*.tsx' --include='*.js' 2>/dev/null
  ```
- Confirm `.env` is in `.gitignore`
- Flag any values that would crash the app if missing

> **Local Xcode builds only:** If the project has a `check:release-env` script, run it. It
> validates that `ios/.xcode.env.local` carries the values a local archive needs, typically
> `NODE_BINARY` plus any service tokens and production URLs the app depends on, and that those
> services are reachable. Skip this for EAS builds,
> where those values are configured as EAS secrets. Consult the project's own docs for the exact
> variable names; they differ per app.

## 6. Run checks

Follow the `check` playbook from the workflows plugin, and report its results.

## 7. Summary

Present a release summary:
- Version: old → new
- Build number: old → new
- Changelog and App Store "What's New" copy (from `release-notes`)
- Any issues found during checks
- Next steps (build with `eas build`, submit to TestFlight, etc.)
