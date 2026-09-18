# SmartPark KE

A functional, web-based parking management system built for the DSA Task One
brief at Multimedia University of Kenya. Companion design document:
`DESIGN.md` (in the repo root, submitted alongside this code).

## What it does

- **Board** (`/`) — live, self-refreshing display of free/occupied bays, so
  drivers can check availability before entering (R1).
- **Entry** (`/entry`) — records a vehicle on arrival, allocates the nearest
  free bay, opens the entry barrier (R2).
- **Exit & pay** (`/exit`) — computes time parked and the fee from the
  tariff table, takes payment (mocked M-Pesa), opens the exit barrier only
  once payment settles (R3, R4).
- **Admin** (`/admin`) — today's revenue, currently-parked count, overstay
  list, a blacklist control, and the gate/audit event logs (M8).

## Run it locally

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Open http://127.0.0.1:5000/ — the database (`smartpark.db`, SQLite) and its
seed data (24 bays across zones A and B, the brief's tariff) are created
automatically on first run.

Try it end to end: check a plate in at `/entry` (e.g. `KDA123A`), watch it
turn amber on the board at `/`, then settle up at `/exit`.

## How the code maps to the design document

| File | Module (DESIGN.md) |
|---|---|
| `modules/structures.py` | The data structures themselves — array/heap/counters (M1), hash-map+linked-list (M4), hash set (M3), sorted array + binary search (M5), FIFO queues (M3/M7), append-only log (M8) |
| `modules/state.py` | Owns the shared structure instances; rebuilds them from the database on startup |
| `db.py` | The database schema from DESIGN.md Section 5.2, on stdlib `sqlite3` |
| `modules/entry.py` | M3 — Vehicle Entry / `CheckIn` |
| `modules/billing.py` | M5 — Fee Computation / `ComputeFee` |
| `modules/payments.py` | M6 — Payment Processing / `SettleAndExit`, behind a swappable `PaymentProvider` |
| `modules/gates.py` | M7 — Gate/Barrier Control / `OpenBarrier` |
| `modules/admin.py` | M8 — Administration & Reporting |
| `app.py` | Flask routes wiring the above to the web UI |
| `templates/`, `static/` | M2 — the availability board and the rest of the UI |
| `seed.py` | First-run seed data (slots, tariff from the brief) |

## Known simplifications (see DESIGN.md's Assumptions Register)

- Payment is a mock M-Pesa provider that always succeeds; swapping in the
  real Safaricom Daraja API only requires a new `PaymentProvider`
  implementation in `modules/payments.py` — nothing else changes.
- Runs single-process, which is fine for a coursework demo; a production
  deployment would move the in-memory structures behind a shared store
  (e.g. Redis) if scaled to multiple worker processes.
- Only one vehicle class (`STANDARD`) is seeded; `slot_type` and
  `tariff_tier` are already schema-ready for `MOTORCYCLE` / `LARGE` /
  `ACCESSIBLE` if the client asks for them later.
