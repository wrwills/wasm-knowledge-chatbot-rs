# AGENTS.md — wasm-knowledge-chatbot-rs

## Dev commands

| Action | Command |
|--------|---------|
| Dev server (hot reload) | `npm run dev` |
| Build prod | `npm run build` |
| Prod build (GitHub Pages) | `trunk build --config Trunk.Dep.toml` |
| Build CSS only | `npm run build-css` |
| Non-WASM tests | `cargo test` |
| WASM tests (headless Firefox) | `npm run test:wasm:headless` |
| Format | `leptosfmt src/` (not rustfmt; see `.vscode/settings.json`) |

## Pitfalls

- **Nightly may break** — very new nightlies can break Leptos dep `tachys` (e.g. `unsized_const_params requires adt_const_params`). Run `cargo update` to pull compatible patch releases of Leptos & tachys. If that doesn't help, pin a known-good nightly date in `rust-toolchain.toml`.

- **`rust-toolchain.toml` pins nightly** — CI installs nightly for `cargo check`, stable for trunk build.
- **`Cargo.lock` is in `.gitignore` but is tracked** — it was committed before the gitignore entry was added. Keep it committed.
- **`csr` feature + `nightly` crate feature** required on Leptos 0.8. rust-analyzer config (`settings.json`) passes `--features csr` and targets `wasm32-unknown-unknown`.
- **CSS pipeline must run before Trunk**: `build-css` (Tailwind PostCSS) writes `public/output.css`, then Trunk picks it up via `<link data-trunk rel="css" href="public/output.css">`.
- **Dev config is separate**: use `trunk serve --config trunk-dev.toml` for debug symbols, no filehash, `public_url = "/"`. Default `Trunk.toml` uses `public_url = "/wasm-knowledge-chatbot-rs/"` (matches GitHub Pages repo name).
- **`.knowledge_cache.json`** (123 KB) is a prebuilt knowledge graph cache committed to the repo — do not delete.

## Architecture

- **Entry**: `src/main.rs` → mounts `<App/>` → `src/lib.rs::App` → `components/main_interface.rs::MainInterface`
- **CSR-only SPA** — no server rendering or hydration. All state lives in `src/state/` modules.
- **Features** (`src/features/`): `crm`, `graphrag`, `webllm`. Feature-specific state, UI tests, and sub-components live alongside them.
- **WebLLM**: loaded as ES module from CDN in `index.html`, bound via `src/webllm_binding.rs`. Model init progress uses JS callbacks.
- **Icons**: Lucide CDN loaded in `index.html`; re-initialized after DOM mutations via MutationObserver. Rust-side binding in `utils/icons.rs`.

## CI / Deploy

- `.github/workflows/ci.yml`: `cargo check` (nightly) + trunk build (stable) → copies `dist/` → `docs/` → GitHub Pages.
- `trunk build --release` outputs to `dist/`; CI copies to `docs/` for Pages. `Trunk.Dep.toml` does this directly.
