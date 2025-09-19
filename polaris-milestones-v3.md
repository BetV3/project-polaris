# Project Polaris — ADHD‑Friendly Milestones (v3 Fully Expanded)

**How to use this:** Work in 10–25 minute boxes. Pick max 3 per session. If stuck >10 minutes, write a one‑line blocker in *Parking‑Lot* and switch tasks. End every session with a commit and an updated checklist.

**Daily 90‑minute loop**: Plan 10 → Build 70 (25/5/25/5/15) → Wrap 10.  
**Guardrails**: 30/60 rule, ≤3 TODOs per file, MVP DoD = clean clone runs, prints example output, README has run steps.

---

## Milestone 0 — Bootstrap & Build Green (T0 sandbox)
**Outcome:** Repo builds locally + CI; skeleton apps exist.

### M0.1 Repo + Conventions
- [ ] Create repo `polaris/` and `README.md` with one‑paragraph overview — *DoD:* README shows a single Build + Run command that works.
- [ ] Add `LICENSE` (MIT) and `CODE_OF_CONDUCT.md` — *DoD:* both present in root.
- [ ] Add `.editorconfig` (UTF‑8, LF, 2/4 spaces) — *DoD:* detected by editor.
- [ ] Add `.gitattributes` (text eol=lf, linguist overrides) — *DoD:* normalization committed.
- [ ] Add `CONTRIBUTING.md` with local build steps — *DoD:* commands copy/paste successfully build.

### M0.2 CMake + Toolchains
- [ ] Top‑level `CMakeLists.txt` (`project(polaris CXX)`, C++20) — *DoD:* `cmake -S . -B build` succeeds.
- [ ] Enable warnings‑as‑errors (non‑MSVC), `-Wall -Wextra -Wconversion` — *DoD:* clean build, no warnings.
- [ ] Add `cmake/` folder + `CMakePresets.json` (`dev`, `release`) — *DoD:* `cmake --list-presets` shows both.
- [ ] Create `apps/hello` with tiny `main()` — *DoD:* `./build/apps/hello/hello` prints “polaris hello”.

### M0.3 CI + Lint
- [ ] Add `.clang-format` (Google base, 120 cols) — *DoD:* `clang-format` produces no diff after run.
- [ ] Add `.clang-tidy` (modernize, readability, bugprone) — *DoD:* `clang-tidy` runs in CI.
- [ ] Add `.gitlab-ci.yml` with `build` + `test` stages — *DoD:* first pipeline is green.
- [ ] (Optional) Pre‑commit hook to format on commit — *DoD:* commit auto‑formats.

### M0.4 Skeleton Tree
- [ ] Create folders `libs/{core,mdp,oms,journal,strat}`, `apps/{md_gen,feed,strat,oms}`, `python/{backtest,console}`, `deploy`, `docs` — *DoD:* tree matches layout.
- [ ] Add placeholder `main()` to each app returning 0 — *DoD:* all four build and run.
- [ ] Commit and tag — *DoD:* `v0.1-bootstrap` exists.

---

## Milestone 1 — Market Data Generator → Feed RX (T0)
**Outcome:** Synthetic ITCH‑like data sent/received with counters.

### M1.1 Message Contract
- [ ] Choose numeric widths (`u8 type`, `u64 seq`, `u64 ts_nanos`, `u64 order_id`, `i32 px_ticks`, `i32 qty`) — *DoD:* field table in `msgs.hpp` comment.
- [ ] Endian doc (little‑endian, packed, no padding/strings) — *DoD:* header block comment.
- [ ] Pack & align (`#pragma pack(push,1)`, `alignof==1`) — *DoD:* unit asserts pass.
- [ ] Encode/decode helpers for one type — *DoD:* fuzz 1k random payloads w/o crash.
- [ ] Golden vectors (hex blobs ↔ structs) — *DoD:* tests validate exact bytes.

### M1.2 Generator (`apps/md_gen`)
- [ ] CLI (`--iface --port --rate --pattern`) via `cxxopts` — *DoD:* `--help` prints flags.
- [ ] Socket opts (`SO_REUSEADDR`, `SO_SNDBUF=8MB`) — *DoD:* opts set, no error.
- [ ] Rate control (token bucket / nanosleep) — *DoD:* ~1k/s ±5% observed.
- [ ] Trend model (`mid += drift + noise`) — *DoD:* CSV plot shows drift.
- [ ] Counters (`msgs_sent_total`, `bytes_sent_total`) — *DoD:* prints each second.

### M1.3 Feed Listener (`apps/feed`)
- [ ] Socket (`SO_RCVBUF=8–16MB`, non‑blocking) — *DoD:* no EWOULDBLOCK spam.
- [ ] epoll loop (edge‑triggered, batch `recvfrom`) — *DoD:* ≥250k msg/s loopback.
- [ ] Parser (type switch, seq monotonic) — *DoD:* bad frames bump `parse_errors`.
- [ ] Metrics (prometheus exposer on `/metrics`) — *DoD:* `msgs_total` visible.
- [ ] Log throttle (rate‑limited logs) — *DoD:* <5 logs/s under load.
- [ ] Tag — *DoD:* `v0.2-feed_rx` pushed.

---

## Milestone 2 — Order Book v0 (T0)
**Outcome:** Level‑2 snapshots from tape.

### M2.1 Price + Levels
- [ ] Tick math (`double↔ticks`, half‑away‑from‑zero rounding) — *DoD:* ≤1‑tick error over 1k tests.
- [ ] Depth arrays (fixed `[10]` per side, sentinel px=MIN, qty=0) — *DoD:* ctor zeroed, tests green.
- [ ] Shift ops (insert at idx, shift tail, drop overflow) — *DoD:* 11th level drops as expected.
- [ ] Read‑only view (`to_view()`) for strategies — *DoD:* strategies compile against the view.

### M2.2 Tape→Book
- [ ] Apply add (locate level, inc qty, keep order) — *DoD:* snapshot matches scripted seq.
- [ ] Apply trade (reduce qty, collapse empty levels) — *DoD:* never negative qty.
- [ ] Snapshot print (ASCII top‑10 both sides) — *DoD:* screenshot saved in `docs/`.
- [ ] Tag — *DoD:* `v0.3-book` pushed.

---

## Milestone 3 — Strategy Stub + OMS Stub (T0)
**Outcome:** Book→Strategy→OMS dry run.

### M3.1 Strategy Interface
- [ ] Header with `onBook`, `onFill`, `start`, `stop` — *DoD:* apps/strat compiles.
- [ ] Config struct (`spread_ticks`, `size`, `inventory_cap`) — *DoD:* YAML → struct.
- [ ] MM0 (±X ticks around mid at fixed size) — *DoD:* >0 orders in 60s sim.

### M3.2 Lock‑Free Queue + OMS Stub
- [ ] SPSC ring (cache‑line padding, pow2 mask) — *DoD:* >1M msg/s microbench.
- [ ] OMS ACK (echo `cl_ord_id`, gen `ord_id`) — *DoD:* correlation ok.
- [ ] Kill switch (atomic flag + SIGUSR1) — *DoD:* cancels all, refuses new.
- [ ] Tag — *DoD:* `v0.4-e2e-dry` pushed.

---

## Milestone 4 — Observability v0 (T0)
**Outcome:** Metrics + dashboard.

### M4.1 Metrics Hooks
- [ ] Counters (feed/strat/oms: msgs, orders, acks, rejects) — *DoD:* curl shows nonzero.
- [ ] Histograms (OMS latency buckets) — *DoD:* p50/p90/p99 graphed in Grafana.

### M4.2 Grafana + Prometheus
- [ ] Compose file (pinned images, volumes) — *DoD:* services healthy.
- [ ] Prometheus scrape jobs (feed/strat/oms) — *DoD:* targets=UP.
- [ ] Dashboard Hot Path (3 panels + single‑stat) — *DoD:* JSON exported to `docs/`.
- [ ] Tag — *DoD:* `v0.5-obs` pushed.

---

## Milestone 5 — Exchange Simulator v1 (T1)
**Outcome:** Price‑time matching; OUCH‑in / ITCH‑out.

### M5.1 Matching Core
- [ ] Order model: `struct Order{u64 id; Side side; i32 px; i32 qty; u64 ts; u64 owner;}`; `enum Side{Buy,Sell}` — *DoD:* header + Doxygen comments.
- [ ] Price levels: map `px→FIFO` per side; store remaining qty — *DoD:* enqueue/dequeue tests pass.
- [ ] Insert path: on **new**, push into side queue; try cross vs best‑opposite while price crosses and qty>0 → create fills — *DoD:* simple cross test passes.
- [ ] Matching rule: better price first; within price, earliest `ts` first — *DoD:* equal‑price orders fill by arrival.
- [ ] Partial fills: decrement resting, emit `Trade` events, continue until done — *DoD:* qty never negative; residue correct.
- [ ] Self‑trade prevent (opt.): if `owner` matches, cancel/skip or decrement both per config — *DoD:* prevention test passes.
- [ ] Cancel flow: locate by `id`, remove from queue; emit cancel event — *DoD:* O(1) with index or documented O(n); tests included.
- [ ] Replace flow: `R` = cancel+new with new px/qty; preserve client id — *DoD:* old removed, new present.
- [ ] Best bid/ask cache: maintain top prices — *DoD:* O(1) read verified.
- [ ] Event bus: `Trade{buy_id,sell_id,px,qty,ts}`, `BookDelta{side,px,qty}` — *DoD:* emitted on every change.
- [ ] Test suite: 20 scenarios (price improvement, partials, queue order, cancels, replaces, crossed book) — *DoD:* `ctest` green.
- [ ] Microbench: 1M scripted orders; record ns/op — *DoD:* baseline numbers in README.

### M5.2 OUCH‑like Ingress
- [ ] Wire schema: 1‑byte type + fixed fields for N/C/R — *DoD:* table in `docs/protocols.md`.
- [ ] Decoder: fast unchecked + slow checked paths — *DoD:* 10k‑case fuzzer, 0 crashes; coverage saved.
- [ ] TCP server: `SO_REUSEPORT` optional; accept one client — *DoD:* sample client sends `N`.
- [ ] ACK/NACK: `R01 price`, `R02 qty`, `R03 throttle` — *DoD:* integration asserts codes.
- [ ] Backpressure: bounded queue; if full → NACK throttle — *DoD:* fill queue test sees NACKs.

### M5.3 ITCH‑like Egress
- [ ] Publisher: UDP with `IP_TTL=1`, batch send — *DoD:* ≥5k msgs/s on loopback.
- [ ] Snapshot cadence: `--snapshot-ms` configurable — *DoD:* periodic snapshots seen by feed.
- [ ] Gap handling: seq increments; refresh request (stretch) — *DoD:* behavior documented.
- [ ] Tag — *DoD:* `v0.6-exsim` pushed.

---

## Milestone 6 — Risk + Journal v1 (T1)
**Outcome:** Pre‑trade checks + binary journal of all traffic.

### M6.1 Risk Rules
- [ ] Schema validation: load YAML, validate ranges — *DoD:* invalid config exits clearly.
- [ ] Exposure tracker: per‑symbol position + notional — *DoD:* updated on new/ack/fill/cancel.
- [ ] Check order: notional → qty → cancel_rate → pos_limit — *DoD:* unit tests hit each path.
- [ ] Audit log: reason + parameters CSV — *DoD:* lines appear on violations.

### M6.2 Journal Writer
- [ ] Frame format: magic, ver, flags, ts, src, type, len, payload, rolling SHA256 — *DoD:* hexdump matches spec.
- [ ] Writer API: open/append/flush/close (`O_DIRECT` optional) — *DoD:* fdatasync test passes.
- [ ] Tap points: feed, strategy, OMS hooks — *DoD:* journal size >0 after 60s run; index created.
- [ ] Checker tool: validates frames & hash; prints counts — *DoD:* OK on fresh; fails on truncated.
- [ ] Tag — *DoD:* `v0.7-oms_risk` pushed.

---

## Milestone 7 — Real Strategies MM1 + TWAP1 (T1)
**Outcome:** Two parameterized strategies ready for sim.

### M7.1 MM1
- [ ] Params parse: YAML → struct; defaults applied — *DoD:* missing values filled.
- [ ] Quote calc: mid + `skew_coeff*inventory` — *DoD:* debug print for 10 ticks.
- [ ] Inventory caps: enforce |inv|≤cap — *DoD:* 10‑min sim bounded.
- [ ] Requote rules: on cross/large move, CXL/REP within cancel‑rate limits — *DoD:* limits respected.

### M7.2 TWAP1
- [ ] Schedule: split parent qty; jitter% — *DoD:* schedule table printed.
- [ ] Throttle: if shortfall, increase aggressiveness within bounds — *DoD:* adapts under low fills.
- [ ] Safety: end catch‑up but under risk limits — *DoD:* caps respected in tests.
- [ ] Tag — *DoD:* `v0.8-strats` pushed.

---

## Milestone 8 — Replay & Python Backtester (T1→T2)
**Outcome:** Deterministic re‑feed + parameter sweeps.

### M8.1 Replay Core
- [ ] Index: write `<offset,ts>` every K frames — *DoD:* binary index produced; random seek works.
- [ ] Determinism: fixed RNG seed; journal ts as clock — *DoD:* byte‑identical outputs.
- [ ] API: `replay(start_ns,end_ns)` — *DoD:* counters match original segment.

### M8.2 Python Backtester
- [ ] Pybind11: expose `run_backtest(params,journal)` — *DoD:* returns metrics or writes Parquet.
- [ ] Metrics: PnL, max DD, Sharpe, hit rate — *DoD:* CSV in `artifacts/metrics.csv`.
- [ ] Plots: equity curve + inventory heatmap — *DoD:* PNGs saved.
- [ ] Sweep: grid over params; save best N — *DoD:* `best_configs.json` created.
- [ ] Tag — *DoD:* `v0.9-backtest` pushed.

---

## Milestone 9 — Trader Console (T2)
**Outcome:** Ops UI for start/stop and parameters.

### M9.1 TUI/App
- [ ] Choose Textual vs Streamlit — *DoD:* decision + why in README.
- [ ] Live view: pull Prometheus HTTP, render orders/s, acks/s, rejects/s — *DoD:* ≥1Hz updates.
- [ ] Param form: typed fields with min/max — *DoD:* invalid values blocked.
- [ ] Actions: start/stop/kill‑switch — *DoD:* control plane logs confirm.
- [ ] Tag — *DoD:* `v1.0-console` pushed.

---

## Milestone 10 — Time & Tuning (T2)
**Outcome:** Latency budget met; clocks synced.

### M10.1 Time Sync
- [ ] Chrony: configure servers; `makestep` — *DoD:* `chronyc tracking` offset <100µs.
- [ ] PTP (if NIC): `ptp4l` + `phc2sys` — *DoD:* `pmc GET TIME_STATUS_NP` within target.

### M10.2 CPU/NIC
- [ ] CPU: `isolcpus`, `rcu_nocbs`, pin procs — *DoD:* documented in `docs/tuning.md` + applied.
- [ ] NIC: disable GRO/LRO; `ethtool -G` rings; IRQ affinity — *DoD:* before/after p50/p99 chart in `docs/`.
- [ ] Tag — *DoD:* `v1.1-tuned` pushed.

---

## Milestone 11 — Soak & Chaos (T3)
**Outcome:** Stability under loss/jitter; no leaks.

### M11.1 Soak
- [ ] Memory watch: RSS + jemalloc stats — *DoD:* graph flat across 2h.
- [ ] Error budget: SLO (<1% non‑risk rejects) — *DoD:* report shows SLO met.

### M11.2 Chaos
- [ ] `tc netem`: 0.5% loss, 2ms jitter script — *DoD:* `deploy/chaos.sh` works.
- [ ] Graceful degrade: lower size / widen spread — *DoD:* metrics reflect adaptation; no crash.
- [ ] Tag — *DoD:* `v1.2-stable` pushed.

---

## Milestone 12 — Ops Kit, Docs & Demo (T3)
**Outcome:** Recruiter‑ready package.

### M12.1 Runbooks & Postmortem
- [ ] Runbook: start/stop, health URLs, common errors, kill‑switch steps — *DoD:* colleague runs w/o help.
- [ ] Postmortem: template + one filled example — *DoD:* committed.

### M12.2 Demo Script
- [ ] Storyboard: 8–10 scenes with expected numbers — *DoD:* dry‑run within ±10%.
- [ ] Artifacts: export dashboard PNGs + sample journal — *DoD:* `artifacts/` populated.
- [ ] Tag — *DoD:* `v1.3-demo` pushed.

---

## Issue/Label Templates
- Title: `feat(feed): epoll UDP listener; log rx rate` / `feat(engine): price-time matching`
- Branch: `feat/area-task`, `fix/area-bug`, `perf/area-tuning`
- Labels: `T0`, `T1`, `T2`, `T3`, `perf`, `risk`, `infra`, `python`, `ux`

**Parking‑Lot:** DPDK userspace NIC, FIX module, hardware timestamps, C# console.
