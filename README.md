# Tickbook

A limit order book and market-data replay engine in C++20, built on Nasdaq order-level data from [LOBSTER](https://lobsterdata.com).

Tickbook replays order events and applies them to a per-price limit order book. Its book state is checked against LOBSTER's reconstructed `orderbook.csv`.

> **Status: work in progress.** See [Status](#status) for exactly what works today.

---

## Data

Tickbook uses LOBSTER sample data, which LOBSTER derives from Nasdaq's ITCH feed. Each symbol-day has two CSV files:

- `*_message.csv`: the event stream. Columns: time, event type, order ID, size, price, direction. Prices are in dollars x 10000.
- `*_orderbook.csv`: the book state after every message. Columns: ask price, ask size, bid price, bid size for each of N levels.

LOBSTER event types:

| Type | Meaning |
|------|---------|
| 1 | New limit order |
| 2 | Partial cancellation |
| 3 | Deletion |
| 4 | Execution of a visible limit order |
| 5 | Execution of a hidden limit order |
| 6 | Cross trade |
| 7 | Trading halt |

The data is **not included** in this repo. Download a free sample from LOBSTER and respect their terms. This project does not parse raw binary ITCH.

---

## Status

Mark each item honestly. Delete this note and any unchecked items you don't plan to build.

**Replay**
- [ ] Reads `message.csv`
- [ ] Reads `orderbook.csv` for validation
- [ ] Type 1: new limit order
- [ ] Type 2: partial cancellation
- [ ] Type 3: deletion
- [ ] Type 4: visible execution
- [ ] Type 5: hidden execution
- [ ] Types 6 and 7: cross trade, trading halt

**Order book**
- [ ] Per-price bid/ask levels maintained as events are applied
- [ ] Best bid / best offer queries
- [ ] Top-N depth queries

**Quality**
- [ ] Book compared against `orderbook.csv` after [every message / sampled checkpoints]
- [ ] Unit tests
- [ ] Benchmarks (see [Benchmarks](#benchmarks))

---

## Build

Requirements: [compiler and version, e.g. GCC 12+ or Clang 15+], CMake [version], Linux.

```bash
git clone https://github.com/Akshaay7705/tickbook.git
cd tickbook
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

## Usage

```bash
./build/tickbook <path/to/SYMBOL_message.csv> <path/to/SYMBOL_orderbook.csv>
```

[Describe what it prints: events processed, validation result, timing summary.]

---

## Design

Describe only choices you have actually made.

- **Parsing:** [e.g. reads the CSV in large chunks, parses fields without per-line allocation]
- **Order lookup:** [e.g. hash map from order ID to order]
- **Price levels:** [e.g. ordered map / flat array indexed by price tick / sorted vector]
- **Memory:** [e.g. preallocated order pool, or "uses standard containers; allocation is a known cost"]
- **Threading:** [e.g. single-threaded by design]

### Layout

```
src/        replay and order book
include/    public headers
tests/      unit tests
bench/      benchmark harness
```

Update this to match the repo.

---

## Validation

The book is rebuilt from `message.csv` and compared to `orderbook.csv` [after every message / at N checkpoints] on [symbol, date, N levels].

Result: [e.g. matches at all levels for N messages / known mismatches: describe them].

```bash
ctest --test-dir build
```

---

## Benchmarks

Report only numbers you have measured, with the method.

| Item | Value |
|------|-------|
| Machine | [CPU model, cores, RAM] |
| OS / compiler | [e.g. Ubuntu 22.04, GCC 12, -O2] |
| Input | [symbol, date, number of messages] |
| Throughput | [X] M messages/sec |
| Per-message latency | p50 [X] ns, p99 [X] ns |
| How measured | [e.g. steady_clock around the apply step, data already in memory] |

Notes: replaying from memory measures book updates only, not file I/O or CSV parsing unless stated. Results vary with hardware.

---

## Limitations

Keep or delete depending on what is true:

- Replays recorded LOBSTER data; does not connect to a live feed or parse raw ITCH.
- Does not model [event types you skipped].
- Single-threaded.
- Reconstructs the book from exchange events; it is not a matching engine.

## Roadmap

- [ ] [next thing you will build]
- [ ] [e.g. parse raw ITCH 5.0 binary directly]
- [ ] [e.g. lower-allocation hot path, profiling with perf]

## References

- LOBSTER data documentation (lobsterdata.com)

## License

[MIT / your choice]
