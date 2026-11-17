# release-readiness

An agent skill for [Claude Code](https://claude.com/claude-code) that checks whether an app is actually ready to ship and writes the ranked list of what's left.

I made it while getting an app ready for its 1.0. The release checklist in the repo was two weeks out of date in both directions: it listed blockers that were already fixed, and it missed that the build attached to the release was twenty commits behind. So the skill doesn't trust any checklist. It runs the tests, builds the app and asks the App Store Connect API what is really set, then walks the app the way a stranger would on a fresh install.

## Install

With the [skills CLI](https://skills.sh), which works for Claude Code, Cursor, Codex and others:

```bash
npx skills add ChrisThoma/release-readiness
```

Or clone it into your Claude Code skills folder:

```bash
git clone https://github.com/ChrisThoma/release-readiness.git ~/.claude/skills/release-readiness
```

For a single project, clone it into `.claude/skills/release-readiness` inside the repo instead.

## Use

In the app's repo, run `/release-readiness`, or ask something like "what's left before we can release?" or "is this ready for the App Store?"

The skill works in four steps. First it checks the facts at the latest commit: the test count, a release build, and which build is attached to the release in App Store Connect. Then it lists what blocks submission, such as missing territories, missing reviewer notes or screenshots older than the UI. Next it installs the app fresh and goes through the first ten minutes as a new user would, looking at defaults, empty states, permission prompts and copy. Last it writes one report: what's verified healthy, the blockers, the polish fixes ranked by how early a new user would hit them, and a short list worth fixing before you submit.

It doesn't change anything until you pick from that list. iOS and the App Store get the most detail. Web and Google Play releases get the same checks in a shorter form.

See [SKILL.md](SKILL.md) for the full method.

## License

[MIT](LICENSE)
