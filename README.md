# Theater Runtime

Theater Runtime is an experimental, neutral-branded browser host for the
open-source Tiberian Dawn simulation. It runs the C++ gameplay core as
WebAssembly in a dedicated worker, presents the battlefield with WebGL2, and
ships as an offline-capable React/Vite PWA.

This is an independent modification first identified as modified on
2026-07-10. Electronic Arts has not endorsed and does not support it.

## What works

- Real Tiberian Dawn simulation at a deterministic 15 Hz with a 60 Hz browser
  presentation loop.
- WebGL2 battlefield rendering, camera pan/zoom, selection, contextual orders,
  production, placement, repair, sell, superweapons, control groups, saving,
  offline recovery, touch controls, and first-battle onboarding.
- Zero-install classic-freeware bootstrap for the 25-mission GDI catalog. The
  generated package is hash-verified, music-free, movie-free, and kept out of
  source control.
- Deterministic release acceptance for GDI Missions 1–5, including all Mission
  4 and Mission 5 variants.
- Optional local conversion of content from a user-owned Remastered
  Collection installation. This is not required for the freeware path.

Mission 6+, the Nod freeware release profile, enhanced replacement art/audio,
semantic radar, broader state deltas, interpolation, gamepad support, and
screen-reader order parity remain in progress. The exact boundary is tracked
in [implementation status](docs/implementation-status.md).

## Repository boundary

This repository owns the browser product:

- `browser/engine`: stable C ABI, deterministic host, and WebAssembly bridge.
- `web`: WebGL/PWA runtime and browser acceptance.
- `tools/content-packer`: verified content conversion and packaging.
- `scripts` and `docs`: policy, release, staging, and acceptance tooling.

The portable engine is pinned as the
[`vendor/vanilla-conquer`](vendor/vanilla-conquer) submodule. The dependency
exposes a reusable Tiberian Dawn remaster-core target; all browser-specific
toolchain flags, exports, compatibility shims, and UX stay here.

Clone with the exact engine revision:

```sh
git clone --recurse-submodules https://github.com/RodimusGPT/cnc-web.git
cd cnc-web
```

If the repository was cloned without submodules:

```sh
git submodule update --init --recursive
```

## Run locally

Prerequisites are CMake 3.25+, Ninja, the Emscripten version in
[`emscripten-version.txt`](emscripten-version.txt), the Node version in
[`.node-version`](.node-version), Corepack, and the Rust toolchain in
[`rust-toolchain.toml`](rust-toolchain.toml).

Build the WebAssembly engine:

```sh
source /path/to/emsdk/emsdk_env.sh
cmake --workflow --preset web-td
```

Install the web dependencies and run the development server:

```sh
cd web
corepack pnpm install --frozen-lockfile
corepack pnpm dev
```

The source-only synthetic demo runs without game content. To build the
zero-install freeware experience into the production directory:

```sh
cd ..
./scripts/build-classic-freeware.sh web/dist
cd web
REQUIRE_BROWSER_ENGINE=1 corepack pnpm build
corepack pnpm preview
```

The freeware builder downloads a pinned source archive, verifies its byte
length and SHA-256, converts the approved files, rejects music/movie content,
and writes only ignored generated output. See
[classic-freeware deployment](docs/classic-freeware.md) and
[the content policy](docs/content-policy.md).

## Verify

Run the asset-free repository gates:

```sh
./scripts/check-no-retail-content.sh
./scripts/tests/test-owned-content-acceptance.sh
PYTHONDONTWRITEBYTECODE=1 \
  python3 -m unittest scripts/tests/test_verify_owned_content_package.py

cmake --workflow --preset testing

cargo fmt --manifest-path tools/content-packer/Cargo.toml --all -- --check
cargo clippy --locked --manifest-path tools/content-packer/Cargo.toml \
  --all-targets -- -D warnings
cargo test --locked --manifest-path tools/content-packer/Cargo.toml --all-targets

cd web
corepack pnpm typecheck
corepack pnpm test
corepack pnpm test:classic-freeware:deployment-tools
corepack pnpm build
corepack pnpm test:bundle-budget
corepack pnpm check:bundle-budget
```

After building the engine, `node scripts/smoke-web-engine.mjs` exercises its
public lifecycle ABI. The complete generated-freeware release gate is:

```sh
./scripts/build-classic-freeware.sh web/dist
cd web
corepack pnpm test:classic-freeware:release
```

That suite covers real engine startup, package preflight, performance,
save/load and offline recovery, Missions 1–5 and their variants, and the
desktop browser matrix. Physical Safari/mobile and human play remain staging
release gates.

## Content and licensing

No EA game data belongs in this repository. Do not commit art, music, speech,
movies, maps, localized text, MEG/MIX archives, or generated `.cncweb`
packages. Optional user-created packages stay in browser-private storage.

Source and distributed WebAssembly are governed by GNU GPL v3 and the
additional terms in [License.txt](License.txt). See [NOTICE.md](NOTICE.md) and
the in-app [legal notice](web/public/legal.html) for provenance, trademarks,
and content-policy details.

Key technical documents:

- [Browser architecture](docs/browser-architecture.md)
- [Performance gates](docs/browser-performance-gates.md)
- [Static-host staging contract](docs/staging-deployment.md)
- [Owned-content acceptance](docs/owned-content-acceptance.md)
