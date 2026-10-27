# Browser CI provenance

The non-publishing `browser-tests.yml` workflow uses Node 24 because the pinned
OpenClaw development dependency requires Node >=24.16 for the full test suite.
The production Dockerfiles remain on the package's supported Node 22 runtime and
install production dependencies only. Other pre-existing workflows are unchanged
by this hardening pass.

## Action pins

Verified against the official repositories' GitHub tag references and release
metadata on October 27, 2026:

| Action | Release | Immutable commit |
| --- | --- | --- |
| `actions/checkout` | `v4.4.0` | `11d5960a326750d5838078e36cf38b85af677262` |
| `actions/setup-node` | `v4.4.0` | `49933ea5288caeca8642d1e84afbd3f7d6820020` |

Sources: `https://api.github.com/repos/actions/checkout/git/ref/tags/v4.4.0`
and `https://api.github.com/repos/actions/setup-node/git/ref/tags/v4.4.0`.
The moving `v4` references resolved to those same commits. The official `v3`
references were also checked; this workflow does not use any v3 actions.
Keep the readable release comments when updating a full-SHA pin.

## Camoufox archive integrity

The official release API **does publish SHA-256 digests** for all six beta.30
archives. `lib/camoufox-integrity.js` pins those values from:

`https://api.github.com/repos/daijro/camoufox/releases/tags/v152.0.4-beta.30`

The release was published September 1, 2026. Each asset's `digest` field is
`sha256:<hex>`. The npm bundled fetcher and both Dockerfiles hash the downloaded
archive before extraction, reject a mismatch, and fail closed for unknown asset
names. The browser workflow exercises that fetcher via `npm run fetch-bin`.
Version-file, executable-version and `ldd` checks remain separate compatibility
checks, not substitutes for archive integrity.

To update the browser release, retrieve its official release metadata, review
and copy each published asset digest, update the bundled release/defaults, and
rerun the integrity and release-consistency tests. An arbitrary VERSION/RELEASE
override will now fail until its upstream-published digest is reviewed and pinned.
Do not replace a missing upstream digest with a locally computed value and call
it verified provenance.

These hashes authenticate downloaded bytes against GitHub's release metadata;
they are not an independently signed publisher attestation or a reproducible
source-build proof. Existing external executable bundles and already-installed
browser caches are not retroactively verified. Makefile/PowerShell prefetch
archives are not used by the current Dockerfiles; the image build downloads and
verifies its own archive. Add-ons, GeoIP data, base images and OS package mirrors
are outside this bounded Camoufox archive check.
