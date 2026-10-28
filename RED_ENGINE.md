# sol-156: Red's CEF engine

This branch starts at CEF 156 upstream commit
`4fa4ab516dacf6c49fc7e6e70d2b7ebce29fa35f`, with Chromium
`156.0.8078.48` (`d0c02bfad08ec92a6ffcf989e53cd995e6381a4c`).
See RED_ENGINE.json for source provenance. The existing
`fix-dynamic-web-accessible-resources` branch is retained separately.

## Layout

Fourteen feature commits contain Red's CEF changes. Seven Chromium patches
are registered at the end of `patch/patch.cfg` under `red_*` names, after the
stock CEF patches they extend. They use CEF's native `-p0` patch format.
Do not apply the old external Red patch stack over this branch.

The features cover extension window identity/action/navigation and resource
redirects, audible/media-session callbacks, overlay scrollbars, captured-tab
mute, Chrome bubbles, child-window close handling, PiP volume, translucent
windows, opaque page backgrounds and the live Link Peek container.

The native document remains full-size. Peek scale, rounded clip and veil
belong to a compositor-only child below NativeViewHost; transforming the
View layer itself would feed scale/offset back into document layout.

## Local builds

Build on the owner's Windows PC with the standard CEF build tooling.
No cloud build workflow is added. Red's `scripts/build_cef.ps1` selects this
repository and an exact commit from `cmake/cef-source.lock.json`.
The source lock is separate from Red's released binary archive lock.

Use CEF automation with `--url=https://github.com/VelikiySebas/cef.git`,
`--checkout=<exact-sol-156-commit>` and `--branch=8078`. The CEF source commit
determines the compatible Chromium revision through
`CHROMIUM_BUILD_COMPATIBILITY.txt`; never mix arbitrary CEF/Chromium heads.
Generate the experimental API wrappers/hashes with
`python tools/version_manager.py -u` in the full Chromium checkout.
Generated wrappers and untracked API hashes are intentionally not committed.
Build Red against matching headers, wrapper, import library and runtime.

## Updating

Keep `upstream` pointed at `https://github.com/chromiumembedded/cef.git`.
Fetch the supported release branch and prepare an update on a separate branch.
Merge the upstream release changes, resolve source/patch conflicts, and update
the Chromium checkout to the version required by the resulting CEF revision.
For a new Chromium milestone, port the feature commits to its supported CEF
release branch and update the recorded base and compatibility metadata.

Run API generation, local engine/client builds and regression checks before
publishing an update to `sol-156`. Red pins the reviewed exact fork SHA; a
branch moving on GitHub cannot silently change a pinned build. Publish and pin
a new binary archive only after its actual release build succeeds.

## Migration verification

All 18 changed canonical CEF source files match the dev checkout used for the
accepted Peek resize fix, after normalizing line endings. Applying the seven
native Chromium patches over stock CEF's Chromium baseline reproduces all 18
changed Chromium files. No engine behavior was changed by reorganizing them.

The existing dynamic-extension-resource test was retained from commit
`fdb790067f3d1057f3c8b2ce28867dee12099f89` and registered on this older release
base. That imported CEF test has not been run as part of this source migration.
Previous dev verification covered native tests (12), semantic API checks (7),
Peek viewport/edge/entry/return and park/resume, with a separately documented
dev-only shutdown DCHECK. A build of the new fork revision and a release
archive require their own verification; source equivalence is not a release.
