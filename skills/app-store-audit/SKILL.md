---
name: app-store-audit
description: Audit the project for Apple App Store readiness. Checks for common rejection reasons including hardcoded values, missing privacy descriptions, accessibility gaps, and required metadata. Run before preparing a submission.
---

Perform a thorough audit of this project for App Store readiness. Check each category below by reading the relevant files and searching the codebase. Report findings as a prioritized list: blockers (will cause rejection), warnings (may cause rejection), and suggestions (improve chances).

## Checklist

### Privacy & Permissions (blockers)
- **Info.plist usage descriptions**: Every native module that accesses protected APIs needs a usage description string. Check for:
  - `NSCalendarsUsageDescription` or `NSCalendarsFullAccessUsageDescription` (EventKit)
  - `NSPhotoLibraryUsageDescription` (Photos)
  - `NSLocationWhenInUseUsageDescription` (Core Location)
  - `NSMotionUsageDescription` (Core Motion)
  - `NSHomeKitUsageDescription` (HomeKit, if the app shows smart home devices)
  - `NSCameraUsageDescription` (if the device camera is used)
  - `NSLocalNetworkUsageDescription` (if the app reaches devices on the local network)
  - `NSRemindersUsageDescription` or `NSRemindersFullAccessUsageDescription` (EventKit reminders)
- Check `app.json`'s `ios.infoPlist` **and** the committed `ios/<AppName>/Info.plist`. Both must
  agree: `expo prebuild` regenerates the latter, so an `app.json` edit alone does not reach a
  committed `ios/` directory.
- **The reverse matters more.** A usage description present for a capability the app no longer
  ships invites a reviewer question. A capability shipped with no usage description terminates
  the app the first time it asks.
- Verify the description strings are user-friendly (not generic like "This app needs access")

### Hardcoded & Environment Values (blockers)
- Search for hardcoded API keys, tokens, coordinates, or credentials in source files (not just .env)
- Check if `.env` values have sensible fallbacks or if the app crashes without them
- Verify every optional integration degrades gracefully when unconfigured, rather than breaking
  the app or hanging on a spinner that implies progress
- Check for hardcoded location coordinates; location-dependent features should work anywhere
- Look for developer-specific values left in source (names, home addresses, network SSIDs,
  personal calendar or album names) that would make no sense to another user

### App Store Metadata (blockers)
- Check `app.json` for: bundle identifier, version, build number, app name, supported orientations
- Verify the app icon exists and meets size requirements (1024x1024 for App Store)
- Check for a launch/splash screen
- **`privacyPolicyUrl` must resolve.** Apple re-checks it on the live listing, so a 404 there is
  a review problem rather than a broken link. Fetch it and confirm a 200:
  ```bash
  curl -s -o /dev/null -w "%{http_code}\n" -L "$(node -p "require('./app.json').expo.privacyPolicyUrl")"
  ```
- App Store Connect also has a **Support URL** field, separate from the privacy one and equally
  mandatory. It lives only in App Store Connect, not in `app.json`, so it is easy to leave
  pointing at a retired domain. Check it by hand.
- Any contact address the app shows users must be one somebody actually reads

### Copy accuracy (blockers)
- Read the splash screen, onboarding, and any marketing copy in the app, and confirm none of it
  advertises a capability that has been removed. This is the first thing a user sees and the
  easiest thing to forget when a feature is cut.
- The same applies to the privacy policy and terms: describing access the app no longer requests
  is one problem, and failing to describe access it does request is a worse one

### Configuration & Onboarding (warnings)
- Can a new user configure everything the app displays, through onboarding or settings, without
  editing code? Walk the widgets the shipping templates actually place and check each one.
- If any displayed value can only be set in source, flag it. Reviewers expect an app to be
  usable out of the box.

### Code Quality (warnings)
- Run `npx tsc --noEmit` and report any type errors
- Check for `console.log` statements that should be removed
- Look for TODO/FIXME/HACK comments that indicate incomplete work
- Check for unused imports or dead code patterns

### Accessibility (suggestions)
- Check if interactive elements have `accessibilityLabel` props
- Look for color-only information indicators (needs text/icon alternatives)
- Check touch target sizes (minimum 44x44 pt for iOS)

### App Transport Security (warnings)
- Check for any HTTP (non-HTTPS) URLs that would need ATS exceptions
- Local network access, for example to a device streaming on the LAN, may need an exception.
  Flag it rather than assuming the default policy covers it.

### Device Compatibility (suggestions)
- Verify the app handles different iPad sizes gracefully
- Check that the orientation declared in `app.json` matches what the app actually supports, and
  that every orientation it allows is genuinely usable. Declaring landscape-only when a layout
  works in portrait costs users; allowing portrait that renders badly costs a review.
- Look for fixed dimensions that might not scale

## Output Format

Group findings by severity:

**🚫 Blockers** - Must fix before submission
**⚠️ Warnings** - Likely to cause issues in review
**💡 Suggestions** - Would improve the submission

For each finding, include: what the issue is, where in the code it occurs, and a brief recommendation for fixing it.
