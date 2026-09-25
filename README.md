# app-template-flutter

GitHub template repo for the "flutter-app" app type in the app factory
(`decisions/0001-app-factory.md`, company-brain). A new Flutter/Play app is created by
using this repo as a GitHub template, then filling in the TODOs below.

## Template changes are never backfilled

**Once an app repo is created from this template, changes made here afterward do not
propagate to it.** This is a snapshot copied once, not a shared dependency. If you find
a bug in a live app that came from this template, fix it in that app's own repo with an
ordinary PR. Only fix it here too if you want *future* apps to get the fix — and if a
template bug has had to be fixed in more than two live apps, that's the signal to
reconsider the template itself (see the ADR's "Consequences" section).

## What's in here

- `release.yaml` — the `release-platform` contract for a Flutter app on Google Play,
  copied from `release-platform`'s
  [`examples/android-only.release.yaml`](https://github.com/ravitejakamalapuram/release-platform/blob/main/examples/android-only.release.yaml).
  Set `package:` to the app's real Android application id before the first release —
  application ids cannot contain hyphens, so it is not filled in from the repo slug
  automatically.
- `.github/workflows/ci.yml` and `.github/workflows/release.yml` — verbatim copies of
  `release-platform`'s `templates/ci-caller.yml` and `templates/release-caller.yml`.
  They only call `release-platform`'s reusable workflows and carry no app-specific
  logic, by design: see each file's own header comment before editing it.
- `fastlane/metadata/android/en-US/` — the Play Store listing text (`release.yaml`'s
  `listing:`), read by `release-platform`'s `listing.yml`. Fill in `title.txt`,
  `short_description.txt` and `full_description.txt` before the app's first listing
  sync; add an `images/` directory (icon, feature graphic, phone screenshots) alongside
  them per fastlane's own layout when those assets exist.
- `CHANGELOG.md` — `release.yaml`'s `release_notes:` source; its first section becomes
  the Play "What's new" text on release.
- `app-metadata.json` — the shape `scripts/gen-products.mjs` (company-brain) reads to
  generate `products/<slug>.md`. Fill in every `TODO`, including `storeId` once the
  Play Console app exists (board action, see below).

## Why no `promote.yml` and no Flutter project here

Promotion (`promote.yml`, staged rollout on the `production` environment) and the
Flutter project itself (`flutter create .`, `lib/`, `android/`) are deliberately not
part of this template. Add `promote.yml` from `release-platform`'s
[`templates/promote-caller.yml`](https://github.com/ravitejakamalapuram/release-platform/blob/main/templates/promote-caller.yml)
when you actually promote a release, and create the `production` environment with
required reviewers first (`release-platform`'s README, "Promoting (Google Play)").
Replace the placeholder entirely with `flutter create .` for the real app: there is no
working placeholder build here the way `app-template-web` ships one, because a real
Flutter project is exactly what `flutter create` already generates correctly, not
something worth freezing into a template that will drift from the Flutter SDK's own
scaffold. **Consequence:** this template repo's own `ci` workflow is expected to fail
(`flutter test`, `flutter build apk --debug`) until an app created from it has run
`flutter create .` — that is normal, not a bug in the template.

## Setting up a new app from this template

1. Fill in every `TODO` in `app-metadata.json`, `release.yaml`, `CHANGELOG.md` and the
   three `fastlane/metadata/android/en-US/*.txt` files. `release.yaml`'s `package:`
   needs a real Android application id (e.g. `com.ravitejakamalapuram.appname`, no
   hyphens).
2. Run `flutter create .` in the new repo to replace the placeholder with a real
   Flutter project, keeping `release.yaml` and the two workflow files.
3. **Board action:** create the Play Console app, then set `storeId` in
   `app-metadata.json` and `package:` in `release.yaml` to its application id.
4. **Board action:** generate the per-repo Android upload keystore
   (`scripts/new-app.sh` prints the exact `keytool` command and the four secret names
   when it creates a `flutter`-type app) and add them under Settings → Secrets and
   variables → Actions.
5. Run `node scripts/gen-products.mjs` in company-brain to generate
   `products/<slug>.md` from `app-metadata.json`.
