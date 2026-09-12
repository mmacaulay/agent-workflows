---
name: app-store-audit
description: Audit the project for Apple App Store readiness. Checks for common rejection reasons including hardcoded values, missing privacy descriptions, accessibility gaps, and required metadata. Run before preparing a submission.
---

Perform a thorough audit of this project for App Store readiness. Check each category below by reading the relevant files and searching the codebase. Report findings as a prioritized list: blockers (will cause rejection), warnings (may cause rejection), and suggestions (improve chances).

## Checklist

### Privacy & Permissions (blockers)
- **Info.plist usage descriptions**: Every native module that accesses protected APIs needs a usage description string. Check for:
  - `NSHomeKitUsageDescription` (HomeKit)
  - `NSCalendarsUsageDescription` or `NSCalendarsFullAccessUsageDescription` (EventKit/Calendars)
  - `NSPhotoLibraryUsageDescription` (Photos)
  - `NSRemindersUsageDescription` or `NSRemindersFullAccessUsageDescription` (Reminders)
  - `NSCameraUsageDescription` (if camera streams are used)
  - `NSLocalNetworkUsageDescription` (if Plex/local network access is used)
- Check `app.json` plugins and `ios/` Info.plist for these keys
- Verify the description strings are user-friendly (not generic like "This app needs access")

### Hardcoded & Environment Values (blockers)
- Search for hardcoded API keys, tokens, coordinates, or credentials in source files (not just .env)
- Check if `.env` values have sensible fallbacks or if the app crashes without them
- Verify Plex integration is optional and doesn't break the app when unconfigured
- Check for hardcoded location coordinates (weather should work for any location)
- Look for Matt-specific values (names, addresses, WiFi SSIDs) that wouldn't make sense for other users

### App Store Metadata (blockers)
- Check `app.json` for: bundle identifier, version, build number, app name, supported orientations
- Verify the app icon exists and meets size requirements (1024x1024 for App Store)
- Check for a launch/splash screen

### Configuration & Onboarding (warnings)
- Is there a settings screen or onboarding flow for users to configure their own:
  - Weather location
  - Guest WiFi credentials
  - Plex server (optional)
  - Which HomeKit home/cameras to display
- If not, flag as a warning — App Store reviewers expect apps to be usable out of the box

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
- Plex local server access may need an ATS exception — flag this

### Device Compatibility (suggestions)
- Verify the app handles different iPad sizes gracefully
- Check if landscape-only orientation is properly enforced
- Look for fixed dimensions that might not scale

## Output Format

Group findings by severity:

**🚫 Blockers** — Must fix before submission
**⚠️ Warnings** — Likely to cause issues in review
**💡 Suggestions** — Would improve the submission

For each finding, include: what the issue is, where in the code it occurs, and a brief recommendation for fixing it.
