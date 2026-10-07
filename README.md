# LightUp build of Shaka Player

This is [LightUp](https://github.com/lightuptw)'s fork of
[shaka-project/shaka-player](https://github.com/shaka-project/shaka-player).
It exists to ship an upstream release plus a small number of patches, as the npm
package `@lightuptw/shaka-player` on GitHub Packages.

This branch (`main`) holds no Shaka code: only this file and the publish
workflow. The code lives on release branches.

## Release branches

One branch per upstream release we build on. A branch starts at upstream's tag
and adds our patches in a straight line, then a release commit. Once a version
is published from it, the branch is never rewritten.

| branch | upstream base | published versions |
|---|---|---|
| `lightup/v5.2.12` | [`v5.2.12`](https://github.com/shaka-project/shaka-player/releases/tag/v5.2.12) | `5.2.12-lightup.1` |

To see exactly what a branch changes:

```sh
git log --oneline v5.2.12..lightup/v5.2.12
```

### Patches on `lightup/v5.2.12`

1. **fix(DASH): Find the segment when rounding misplaces a fixed-duration lookup.**
   `FixedDurationSegmentIndex_.find()` (new in 5.2) divides the time by the
   segment duration and takes that as the segment number. At some boundaries
   (e.g. 56.056 s, 112.112 s, 448.448 s with a 4.004 s segment) floating-point
   rounding puts the result one segment off and the lookup returns null, which
   stalls playback. The fix treats the division as an estimate and moves at
   most one reference back or forward to the last reference starting at or
   before the time, which is what the generic `SegmentIndex.find()` does. Comes
   with a test that compares the fixed-duration index against a plain index at
   every boundary.
2. **build: Accept dot-separated pre-release identifiers in checkversion.**
   Upstream's release check only allowed `-identifier`; our versions are
   `-lightup.N`.
3. **chore: Publish as @lightuptw/shaka-player on GitHub Packages.** Package
   name, registry and repository links.
4. **chore(lightup/v5.2.x): release 5.2.12-lightup.1.** Version in
   `package.json`, the lockfile and `lib/player.js`, and the changelog entry —
   the same files upstream's release commit touches.

## Versions

`<upstream version>-lightup.<n>`, for example `5.2.12-lightup.1`. The upstream
base is readable from the version; `n` counts our releases on that base. As a
semver pre-release it is never matched by a range like `^5.2.0`, so consumers
pin it exactly. Each version has a tag `v<version>` on its release commit.

## Publishing

The **Publish release** workflow on this branch takes a tag and, in order:

1. checks the tag sits in a straight line on upstream's release tag;
2. rebuilds upstream's own tag and requires the files we consume to be
   byte-identical to upstream's npm tarball;
3. builds our tag with upstream's publish gate (`checkversion` + `build/all.py`)
   and packs it once — that tarball is what gets published;
4. runs upstream's browser tests on both builds (Linux Chrome and Firefox) and
   fails on any spec that passes upstream but fails on ours;
5. reports which files in our tarball differ from the upstream rebuild;
6. waits for approval in the `publish` environment, then publishes to GitHub
   Packages and attaches the tarball to a GitHub release.

## Consuming

```json
"shaka-player": "npm:@lightuptw/shaka-player@5.2.12-lightup.1"
```

The alias keeps `import 'shaka-player'`, `shaka-player/dist/shaka-player.ui`
and `shaka-player/dist/controls.css` working unchanged. The package is on
GitHub Packages, so `npm` needs an `@lightuptw` registry entry and a token with
`read:packages`; another repository's workflow also needs read access granted
under the package's "Manage Actions access".

## Next upstream release

Create `lightup/v<new>` from upstream's new tag, cherry-pick patches 1–3
(drop 1 if upstream has merged the fix), add a release commit for
`<new>-lightup.1`, tag it, and run the workflow.
