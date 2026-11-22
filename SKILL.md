---
name: qbit-gate
description: Build correct 1/2/3-qubit logic gate selectors and circuits in Qiskit + Aer Streamlit demos. Use when a quantum gate demo raises CircuitError('duplicate bit arguments'), produces wrong control/target indices, segfaults on Streamlit rerun, or when choosing between st.radio and st.selectbox for qubit selection. Triggers on "quantum gate demo", "CNOT control target", "Toffoli index", "duplicate bit arguments", "Qiskit unitary", "Aer segfault".
---

# Qbit Gate

Build 1/2/3-qubit logic gate demos whose bit indices are mutually exclusive and whose simulator survives Streamlit reruns.

**Core invariant:** gate families unsupported by the qubit count are **not offered at all**; every control / target selector is a visible `st.radio(..., horizontal=True)`; and mutually exclusive indices are enforced by **option filtering at selection time** — never by a post-hoc correction or an `st.info` explanation.

## When to Use

- CircuitError: `duplicate bit arguments` from `qc.cx(0,0)`, `qc.swap(1,1)`, `qc.ccx(0,0,2)`
- Two- or three-qubit gate selector missing a control or target widget
- AerSimulator segfault / memory segment error after a Streamlit rerun
- Raw matrix gate fails with `TypeError: issubclass() arg 1 must be a class`
- Designing the gate-selection half of a Qiskit Streamlit app

## Gate taxonomy (from ~/薦家/布球/ distilled notes)

| Family | Gates | Selector count | Qiskit call |
|--------|-------|----------------|-------------|
| 1 qubit | I, X, Y, Z, H, S, T, Rx(θ), Ry(θ), Rz(θ) | 1 target | `qc.unitary(U, [t])` |
| 2 qubits | CNOT, CZ, SWAP | **2** (control + target) | `qc.cx(c,t)` / `qc.cz(c,t)` / `qc.swap(c,t)` |
| 3 qubits | Toffoli (CCNOT), Fredkin (CSWAP) | **3** (control 1 + control 2 + target) | `qc.ccx(c1,c2,t)` / `qc.cswap(c1,c2,t)` |

Three-qubit gates need exactly 3 distinct indices. Do **not** auto-derive the third bit from the other two — the user must see and control all three radios.

## Rules

### 1. Selector form (discipline)
- Use `st.radio(label, options, horizontal=True)` for **every** selector: qubit count, gate family, gate, control, target.
- Never use `st.selectbox` for these — it hides options and breaks the horizontal-radio convention.
- Never collapse a 3-qubit gate into 2 radios plus an inferred bit.
- **Hide unsupported families instead of warning about them.** Build the family options from `n`:

```python
families = ["Single-qubit"]
if n >= 2: families.append("Two-qubit")
if n >= 3: families.append("Three-qubit")
family = st.radio("Gate family", families, horizontal=True)
```
  With `n == 2` the Three-qubit option does not exist; no `st.warning` is ever needed.

### 2. Mutual exclusion (option filtering, not post-hoc correction)
Filter the options of each later selector so a colliding index **cannot be selected**. Both (all three) radios stay visible; their defaults are distinct by construction, so no explanation message is emitted.

```python
# two-qubit — both radios visible, target excludes control
ctrl = st.radio("Control", list(range(n)), horizontal=True)
tgt  = st.radio("Target", [q for q in range(n) if q != ctrl], horizontal=True)
st.caption(f"Control = q{ctrl} · Target = q{tgt}")   # plain display, NOT st.info

# three-qubit — three radios visible, each excludes all previously chosen bits
c1  = st.radio("Control 1", list(range(n)), horizontal=True)
c2  = st.radio("Control 2", [q for q in range(n) if q != c1], horizontal=True)
tgt = st.radio("Target", [q for q in range(n) if q not in (c1, c2)], horizontal=True)
st.caption(f"Control 1 = q{c1} · Control 2 = q{c2} · Target = q{tgt}")
```

Never show all indices then fix a collision afterward (`if tgt == ctrl: tgt = ...; st.info(...)`). That is the outdated behavior this rule replaces.

### 3. Qiskit 2.x matrix gates
`qc.append(matrix, qubits)` is invalid in Qiskit 2.x — it calls `issubclass()` on an ndarray. Use `qc.unitary`:

```python
U = SINGLE_GATES[gate]          # 2x2 complex ndarray
qc.unitary(U, [target])         # correct
# qc.append(U, [target])        # TypeError: issubclass() arg 1 must be a class
```

Named gates (`qc.x`, `qc.h`, `qc.cx`, …) remain valid.

### 4. Aer survivability across reruns
```python
import os
os.environ.setdefault("OMP_NUM_THREADS", "1")   # BEFORE importing qiskit_aer
...
from qiskit_aer import AerSimulator

@st.cache_resource
def get_simulator():
    sim = AerSimulator()
    sim.set_options(max_parallel_threads=1, max_parallel_experiments=1)
    return sim
```
- Construct the simulator **once**. Rebuilding it per rerun leaks native threads and segfaults.
- `matplotlib.use("Agg")` before `pyplot`, and `plt.close(fig)` after every `st.pyplot(fig)`.
- Aer `get_counts()` keys are already bitstrings (`"00"`). Use `str(k)` — `format(k, "0nb")` raises `ValueError`.

## One complete example

See the working demo at `~/薦家/布球/qbit-gate-viz/app.py`. Its sidebar implements all rules above: with `n == 2` only Single/Two families are offered; Target options exclude Control and the 3-qubit selectors exclude all previously chosen bits. Verified with real-radio AppTest checks — no collision is selectable, so no `st.info` / `st.warning` is emitted.

Startup:
```bash
cd ~/薦家/布球/qbit-gate-viz
streamlit run app.py
```

## Common mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| `qc.append(ndarray, [q])` | `TypeError: issubclass()` | `qc.unitary(ndarray, [q])` |
| Same index for control and target | `CircuitError: duplicate bit arguments` | filter later selector's options with `[q for q in range(n) if q != ctrl]` — no `st.info` |
| Three-qubit family shown when `n == 2` | warning / empty selector | build `families` from `n`; unsupported families never appear |
| `AerSimulator()` inside the render path | segfault after rerun | `@st.cache_resource` + single-thread options |
| `format(count_key, "02b")` | `ValueError: Unknown format code 'b'` | key is already `"00"` — use `str(k)` |
| Figure not closed | memory growth | `plt.close(fig)` |
| `st.selectbox` / hidden third bit | baseline failure | visible `st.radio(horizontal=True)` × N |

## Streamlit rules
Inherits `viz-dboard` Streamlit rules: single `app.py`, no `pages/`, `st.radio` not `st.selectbox`, `st.columns([3,7])`, `width='stretch'`/`'content'`, `with` context, 2D → matplotlib, **3D → PyVista + Panel (including Bloch spheres; no matplotlib-3D exception)**, Streamlit Cloud only for Streamlit apps.

The matplotlib-3D Bloch panel in `~/薦家/布球/qbit-gate-viz/app.py` is **legacy** (it predates this rule and is kept only for the gate-selector demo). For 3D Bloch/evolution use `~/薦家/布球/bloch-evolution-viz/app.py` (`panel serve app.py`), grounded in `5布球態演化維問.py`.

## References
- Distilled notes: `~/薦家/布球/{量子邏輯閘運算.md,總結邏輯閘.md,比特閘Util.py}`
- `qbit-viz` — Bloch sphere / Q-sphere rendering and visualization conventions
- `viz-dboard` — Streamlit rules and demo-tool location conventions
- audit gate: `audit-skill`
- Logic check: 2026-11-22 — top-to-bottom verified: core invariant ↔ Rule 1/2 (dynamic families + option filtering) ↔ example ↔ common mistakes all aligned; no stale `st.info` rule remains.
