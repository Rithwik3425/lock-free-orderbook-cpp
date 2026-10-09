# High-Frequency Matching Engine & Order Book (`orderbook`)

A high-performance, single-threaded financial order book and matching engine written in C++20. This project implements deterministic price-time priority matching with zero dynamic memory allocations on the hot path, benchmarked against standard reference implementations.

---

## Architecture Overview

Production HFT (High-Frequency Trading) matching engines run single-threaded per financial instrument to ensure strict sequence ordering and deterministic matching without lock contention. High throughput is achieved via data locality, cache-friendly structures, and low latency.

```
       [ Incoming Orders / Commands ]
                     │
                     ▼
         ┌──────────────────────┐
         │ Input Command Parser │
         └───────────┬──────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │         Matching Engine Core        │
  │                                     │
  │  ┌──────────────┐  ┌──────────────┐ │
  │  │ Bids (Buys)  │  │ Asks (Sells) │ │
  │  │ Max-Heap/Map │  │ Min-Heap/Map │ │
  │  └──────┬───────┘  └──────┬───────┘ │
  │         └────────┬────────┘         │
  │                  ▼                  │
  │     Price-Time Priority Matcher     │
  └──────────────────┬──────────────────┘
                     │
                     ▼
          [ Event / Trade Stream ]

```

---

## Core Rules & Design Decisions

### Matching & Priority Rules

1. **Price-Time Priority**: Orders match at the best price first. Multiple orders at the same price are matched FIFO based on arrival time.
2. **Resting Orders**: Unmatched limit orders remain in the book until filled or explicitly cancelled. Market orders execute immediately against available liquidity; any unfilled remainder is discarded.
3. **Price Precision**: All prices are specified as integer ticks (e.g., `$100.50` = `10050`) to eliminate floating-point precision errors.

### Edge-Case Policies

* **Market Order with Empty Book (B3)**: Rejected immediately.
* **Modify Quantity (D8)**: Modifying quantity applies strictly to the **remaining** quantity.
* *Decreasing quantity*: Retains queue priority.
* *Increasing quantity / Changing price*: Sent to the back of the price queue.


* **No-Op Modify (D7)**: Modifying an order without changing price or quantity is rejected as a no-op.
* **ID Lifecycle (E4)**: Order IDs must be globally unique. Reusing an ID from a filled/cancelled order is rejected.

---

## Project Structure

```
orderbook/
├── CMakeLists.txt        # Top-level build definitions
├── README.md             # Project overview & documentation
├── NOTES.md              # Optimization log & benchmark notes
├── src/
│   ├── types.hpp         # Domain types (OrderId, Price, Qty, Order, Event)
│   ├── order_book.hpp    # OrderBook interface declarations
│   ├── order_book.cpp    # OrderBook business logic
│   └── main.cpp          # CLI interface and executable entry point
├── tests/                # Unit tests and differential harness
├── testcases/            # Edge-case input and expected-output suites
└── docs/                 # Architectural specifications

```

---

## Getting Started

### Prerequisites

* **macOS** or **Linux**
* **Clang++** (supporting C++20)
* **CMake** (v3.20 or higher)

On macOS, install build dependencies via Xcode Command Line Tools and Homebrew:

```bash
xcode-select --install
brew install cmake

```

### Build & Run

1. **Configure build files**:
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug

```


2. **Compile the executable**:
```bash
cmake --build build

```


3. **Run the program**:
```bash
./build/orderbook

```



---

## Development Roadmap

* [x] **Phase 0: Spec & Design**: Formulate matching rules, edge cases, and output event definitions.
* [x] **Phase 1: Project Setup**: CMake configuration, toolchain setup, and folder structure.
* [ ] **Phase 2: Reference Implementation**: Correctness-first engine using standard containers (`std::map`, `std::list`, `std::unordered_map`).
* [ ] **Phase 3: Differential Test Harness**: Deterministic replay runner with property-based checking and ASan/UBSan coverage.
* [ ] **Phase 4: Optimized Engine Core**: Flat-array level lookups, intrusive linked lists, pre-allocated object pools, and open-addressing hash maps.
* [ ] **Phase 5: Lock-Free Ingest Buffer**: Single-Producer Single-Consumer (SPSC) ring buffer connecting feed handlers to the engine core.
* [ ] **Phase 6: Microbenchmarks**: Latency profile measurement (p50, p99, p99.9) using Google Benchmark.

---

## License

Distributed under the MIT License.
