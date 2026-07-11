# COLLATZ-VERIFIER

**Parallel computational search of the Collatz Conjecture (3n+1), with Lean 4 partial proofs and a WORM + Merkle audit trail for every trajectory.**

`Ω ← TRUST ∧ CODE`

---

## OVERVIEW

`collatz-verifier` is a sovereign-compute node in the SnapKitty constellation
dedicated to the 87-year-old Collatz Conjecture: *every positive integer
eventually reaches 1 under the map `n → 3n+1` (odd) / `n → n/2` (even).*

The repo does not claim to prove the conjecture. It provides **computational
evidence at scale** (a rayon-parallelized Rust engine), **formal partial proofs**
of special cases (Lean 4 + Mathlib), and — the SnapKitty part — **cryptographic
verifiability**: every trajectory is sealed with SHA-256 and folded into a Merkle
tree written to an append-only WORM ledger.

## WHAT IT IS

- A **Rust workspace** with two crates: a computational `engine` and an Axum REST `api`.
- A **WASM-exportable** engine (`compute_trajectory_wasm`, `parallel_search_wasm`) driving a browser visualizer.
- A **Lean 4 proof library** formalizing basic Collatz properties and WORM/Merkle integrity notions.
- A **WORM ledger** (`ledger/`) holding sealed trajectories and the current Merkle root.
- A documented **multi-agent protocol** (ATLAS / TENSOR / LEDGE / AXIOM) describing how work is coordinated, computed, verified, and sealed.

## ARCHITECTURE / COMPONENTS

| File | Language | Purpose |
|------|----------|---------|
| `engine/src/lib.rs` | Rust | Core engine: `collatz_step`, `compute_trajectory`, `parallel_search` (rayon), `find_max_length`, `find_max_value`, `compute_merkle_root`, `seal_to_worm`, plus WASM bindings and a full unit-test suite. |
| `engine/src/main.rs` | Rust | `collatz-search` CLI — searches `[start, end)`, prints stats, computes Merkle root, appends to `ledger/collatz_worm.jsonl`, writes `ledger/merkle_root.txt`. |
| `engine/Cargo.toml` | TOML | Engine crate: `rayon`, `sha2`, `serde`, `wasm-bindgen`; `cdylib`+`rlib`; LTO release profile. |
| `api/src/main.rs` | Rust | Axum server exposing `/verify`, `/search`, `/seal`, `/health` on `127.0.0.1:3000`. |
| `api/Cargo.toml` | TOML | API crate depending on `collatz-engine`, `axum`, `tokio`. |
| `proofs/Collatz.lean` | Lean 4 | `collatz`, `reaches_one`, `collatz_conjecture`, proven lemmas (`even_decreases`, `pow_two_reaches_one`, `double_reaches`, `quad_reaches`, cycle facts) + `merkle_valid`, `worm_immutable`. |
| `proofs/lakefile.lean` | Lean 4 | Lake package pulling in Mathlib. |
| `frontend/index.html` | HTML/CSS/JS | Browser visualizer: trajectory display, range search, WORM audit trail, per-trajectory SHA-256 seals. |
| `ledger/collatz_worm.jsonl` | JSONL | Append-only WORM ledger of sealed trajectories. |
| `ledger/merkle_root.txt` | Text | Current Merkle root (`b5e2d354…bca713`). |
| `COLLATZ.md` | Markdown | Full system design + multi-agent protocol. |
| `metadata.json` | JSON | Provenance, Plasma Gate (Ed25519) enforcement, WORM chain link. |

### Tree

```
collatz-verifier/
├── engine/          # Rust core: parallel search + Merkle + WORM sealing
│   └── src/{lib.rs, main.rs}
├── api/             # Axum REST API (/verify /search /seal /health)
│   └── src/main.rs
├── proofs/          # Lean 4 partial proofs + lakefile
├── frontend/        # HTML/CSS/JS trajectory visualizer
├── ledger/          # WORM ledger + Merkle root
├── theorems/        # theorem index
├── Cargo.toml       # workspace (engine + api)
├── COLLATZ.md       # system design doc
└── metadata.json    # provenance + Plasma Gate seal
```

### Verified computational facts (from the engine test suite)

- `compute_trajectory(6)` → length **9**, max value **16**
- `compute_trajectory(27)` → length **112**, max value **9232**
- `parallel_search(1, 100)` → **99** trajectories, all reach 1

## HOW IT FITS THE CONSTELLATION

- **WORM chain / Bifrost** — every sealed trajectory is an append-only JSONL entry
  with a SHA-256 `trajectory_hash` and a `merkle_proof`, anchored to
  `Bifrost_WORM_Chain_20260710_01`.
- **Merkle / 3-witness verification** — three independent layers must agree before a
  result is trusted: the **Rust engine** recomputes the trajectory, the **Merkle root**
  binds it to the whole batch, and the **Lean proofs** guarantee the underlying
  properties (e.g. powers of two provably reach 1).
- **Plasma Gate / Ed25519** — `metadata.json` marks the node `Ed25519_Enforced` with a
  SHA-256 append-only tamper-evidence chain.
- **P/NP swarm** — search ranges are partitioned as independently verifiable
  work units (the ATLAS → TENSOR → LEDGE → AXIOM flow), each cheap to check.

## BUILD / USAGE / INSTALL

```bash
# Build the workspace (engine + api)
cargo build --release

# Run tests
cargo test

# Parallel search + seal to WORM ledger
cargo run --release --bin collatz-search -- 1 1000000

# Start the REST API → http://127.0.0.1:3000
cargo run --release --bin collatz-api
```

API endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/verify?n=27` | GET | Compute + verify a single trajectory |
| `/search?start=1&end=1000` | GET | Parallel search with Merkle root |
| `/seal?n=27` | POST | Seal a trajectory to the WORM ledger |
| `/health` | GET | Health check |

Verify the Lean proofs (requires Lean 4 + Mathlib):

```bash
cd proofs && lake build
```

Check ledger integrity:

```bash
cat ledger/merkle_root.txt          # current root
wc -l ledger/collatz_worm.jsonl     # entry count
```

## KEY FILES REFERENCE

- **`engine/src/lib.rs`** — the trusted computational core; deterministic seals and Merkle folding make every result independently reproducible.
- **`api/src/main.rs`** — thin HTTP surface over the engine with an in-memory ledger guard.
- **`proofs/Collatz.lean`** — the formal layer; proves the special cases that hold and states the open conjecture honestly.
- **`ledger/`** — the immutable evidence trail.

## LICENSE

Apache-2.0.

---

*Created by Ahmad Ali Parr · SnapKitty Collective · the-49th-call · SNAPKITTYWEST · 2026.*
