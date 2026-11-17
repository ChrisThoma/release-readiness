---
name: release-readiness
description: >-
  Audit whether an app is actually ready to ship, and produce the prioritized
  punch list that gets it there. Verifies real state (builds, tests, store
  config) instead of trusting status docs, then runs a polish pass through a
  stranger's first ten minutes. Works for iOS/App Store first, with hooks for
  web and Android. Trigger phrases: "release readiness", "ready for release",
  "ready to ship", "what's left for launch", "v1 checklist", "prep for
  launch", "app store readiness", "make this a good release".
---

# Release readiness

Answer two questions, in order, and deliver the result as one prioritized punch list:

1. **What blocks shipping at all?** (mechanical: builds, store config, review requirements)
2. **What would make a stranger's first ten minutes feel finished?** (polish: defaults, discoverability, reward, copy)

The deliverable is the punch list: blockers first, then polish ranked by first-impression impact, plus a "verified healthy, don't touch" section so good work doesn't get churned. Don't start fixing until the owner picks from the list, unless they've already asked for fixes.

## Principle: verify, never trust

Status docs, checklists and READMEs describe the world as of the day someone last edited them. Code has moved since. So:

- **Every claim gets re-verified against its source of truth**: the store's API, the actual test run, the actual archive, the actual screen. A checklist item marked done is a *lead*, not a fact.
- **Compare dates, always**: last uploaded build vs. latest commit; screenshots vs. UI-changing commits; listing copy vs. current feature set. Staleness between artifacts is the most common pre-release defect, and no single artifact reveals it.
- **The newest code is the least audited.** Bug hunts and accessibility passes cover what existed when they ran. Find the commits since the last audit or build and give them the scrutiny the older code already got.

## Phase 1: ground truth (parallelize)

Run these as parallel subagents where available; none depends on another.

**Build and test verification (at HEAD):**
- Full test suite. Report the **executed test count**, not just green. A test file missing from the project or build config passes silently by never running. Compare the count to the last known baseline.
- Release build or archive with the real configuration. Capture warnings in first-party code (concurrency warnings become errors on future toolchains, so note them even if they don't block).
- Check the built artifact's metadata: version and build numbers are the intended literals (not unexpanded `$(VARS)`), the commit stamp is present and clean, the bundle or app ID is correct.

**Store and deploy state (through the API where possible):**
- Which build is attached to the pending release, and what commit does it carry? Diff that against HEAD and list every user-visible change that would ship stale.
- Walk the store's own requirements list empirically (see the iOS section below for the concrete API checks). Record what's set and what 404s.

**Doc reality check:**
- Read the repo's own release notes and checklists, then mark each line verified-true, verified-false or unverifiable. Report the differences, and update the doc at the end so the next release starts honest.

## Phase 2: mechanical blockers

Everything on this list either blocks submission outright or gets the release rejected or embarrassed:

- **A final build from HEAD**, attached to the release. If any user-visible commit postdates the attached build, cut a new one (bump the build number where the project defines it, not in generated files).
- **Store configuration is complete**: pricing, territory availability, privacy questionnaire, age rating, categories. Check each directly. These fail as 404s and nulls, not as warnings.
- **Screenshots regenerate from the final build** if the UI changed since they were taken. Automate this if the repo has a screenshot script; note it if it doesn't.
- **Listing copy is still true**: description, subtitle and keywords vs. what the app now does. Feature renames and new modes drift out of listings silently.
- **Reviewer notes** where a human reviewer can't exercise the app unaided (hardware needed, an account needed, mic or camera flows). On iOS these can be set *before* submission through `appStoreReviewDetail`. Set them early, not at the deadline.
- **Permission-denial paths don't dead-end** (App Review actively probes this): every permission the app requests has a graceful denied state with a way out.
- **Legal and compliance**: the privacy policy URL is live, the privacy manifest matches actual data collection, the export-compliance key is set, and bundled assets are licensed.

## Phase 3: polish, the stranger's first ten minutes

Two passes with different instruments. Both rank findings by how early a cold user hits them.

### Hands-on: fresh-install walkthrough

Uninstall first, because first-run state only exists once. Then walk the app as someone who has never seen it, taking screenshots as you go:

- **First screen**: is the default configuration the one a beginner should have? Check every shipped default against the least experienced plausible user. Wrong defaults are the highest-impact, lowest-effort polish fix there is.
- **Onboarding discoverability**: if a guided or intro mode exists, is a user with no data *offered* it at the moment they'd need it, or is it a buried setting? A feature only reachable through settings does not exist for new users.
- **Empty states**: zeros and empty lists on day one. Are they designed ("not seen yet", hidden until earned) or awkward ("Best 0" next to a trophy)?
- **The reward moment**: do the right answer, the streak and the personal best *feel* like anything (animation, haptic, sound)? Does the first session celebrate something unearned, like a "new record" that's only the first data point?
- **Permission prompts**: does the first mic, camera or notification request come with context, or does it interrupt the first real task?
- **Interruption and backgrounding** during the core flow, if your environment can test it.
- **Foldables and other screen shapes**: if the app runs on a foldable like the iPhone Duo, walk every screen closed and open, and screenshot each. A layout that only stretches on the open display, or text cut off on the narrow outer one, is a finding. Do this before the release build is cut, so the fixes make it in. On the Duo simulator, [devicehubctl](https://github.com/ChrisThoma/devicehubctl) can fold the device from a script.

### Code-level audit (a subagent with fresh eyes)

- **Copy voice**: collect the user-facing strings. Flag register drift (formal or passive lines in an app that otherwise uses contractions), a stray first-person "we", footer text that explains a toggle backwards, jargon the target user won't know, and promises the current mode or level can't keep.
- **Bundled content** (any app whose value is its own text: card meanings, a glossary, prompts, activity ideas, lessons). Read every entry, not a sample. Flag lines that don't make sense, signs of AI writing (the [de-ai-text](https://github.com/ChrisThoma/de-ai-text) skill finds them), regional phrasing that doesn't match the app's locale, and the same shape or words repeated across entries. Then have a separate adversarial agent check each entry against its named source (a tarot deck's tradition, music theory, a style guide) and argue against every fix before it's made. The owner approves the changes before the first build. One app that shipped its 1.0 without this pass later found six lines that didn't make sense and twelve entries to reshape, and fixing them took its whole next release.
- **Delight inventory**: haptic, sound and animation coverage on success, failure and milestones. Note what fires and what's flat. An absence may be deliberate, so report it as fact, not as a defect.
- **Finished-app furniture**: review prompt, about and version display, support and privacy links in the app, a share action. List what's present and absent without judging, since some absences are principled (an app with no networking, for example).
- **Accessibility on the newest features**: the older code's accessibility was probably audited; the last few weeks of commits weren't. Every new control or question type needs a workable VoiceOver and TalkBack path and handles large Dynamic Type sizes.
- **Leftovers**: debug prints, TODO and FIXME in shipping paths, dead feature flags, deprecated API warnings.

### Identity check

Look at the icon at home-screen size and next to the app's real UI: contrast, legibility, and whether the icon and the app read as the same product. Do the same for the screenshot set as a whole, since it's the listing's first impression.

## Phase 4: the report

Structure the final report exactly like this:

1. **Verified healthy**: what was checked and passed, so it isn't churned again.
2. **Blockers**: the mechanical list, each with the action that clears it, and a time estimate for the whole set.
3. **Polish, ranked**: numbered, worst first by first-ten-minutes impact, each with file:line and the concrete fix.
4. **Later or optional**: real findings that shouldn't hold up this release (toolchain warnings, unverified niche configurations), so they're recorded and not lost.
5. **A recommended shortlist**: "if you fix N things before submitting", small enough to actually happen.

Close by offering to work through the shortlist. Don't start unasked.

## Platform notes

### iOS / App Store Connect

Query the API directly, with a JWT signed by an App Store Connect API key (`.p8`; `~/.appstoreconnect/private_keys/` is the usual place), or through a command-line client such as [asc](https://asccli.sh) if the project uses one. The high-yield checks, all read-only:

- `GET /v1/apps/{id}/appStoreVersions`: version state (`PREPARE_FOR_SUBMISSION`, and so on).
- `GET /v1/appStoreVersions/{v}/build`: the attached build. Diff its upload date against `git log`.
- `GET /v1/appPriceSchedules/{id}/baseTerritory`: is pricing set at all?
- `GET /v2/appAvailabilities/{id}`: a 404 means **no territories are selected**, which blocks release.
- `GET /v1/appStoreVersions/{v}/appStoreReviewDetail`: null means no reviewer notes or contact. It can be created before submission.
- `GET .../appScreenshotSets` per localization: is each display type's set complete?
- The **App Privacy questionnaire has no API**. It can only be confirmed in the browser; say so rather than guessing.
- The simulator can't exercise mic or camera hardware. Anything that depends on real hardware needs a device pass, or an explicit "unverified on device" flag in the report.

### Web

The sources of truth are the deployed environment and CI, not the repo. Check DNS and TLS, error pages, analytics and consent config, robots and meta tags, Open Graph and social cards against the current design, and that the deployed commit is HEAD.

### Android / Google Play

Same shape, through the Play Developer API: release track state, the attached bundle vs. HEAD, the data safety form, and store listing assets vs. the current UI.
