# qbit-gate

Build correct 1/2/3-qubit logic gate selectors and circuits in Qiskit + Aer Streamlit demos.

Mutually exclusive control/target indices are enforced by **option filtering at selection time** — not by post-hoc correction or `st.info` messages. Gate families unsupported by the qubit count are not offered at all.

## What it does

- Dynamic family options: `n=2` offers only Single/Two-qubit gates
- Mutual exclusion by filtering later selectors' options (Control 1 / Control 2 / Target all visible and distinct)
- Qiskit 2.x `qc.unitary()` for raw matrix gates
- AerSimulator rerun safety: `@st.cache_resource`, `OMP_NUM_THREADS=1`, single-thread options
- matplotlib figure cleanup and Aer bitstring count keys

## Install

```bash
npx skills add phycomp/qbit-gate@qbit-gate -g -y
```

## Usage

Trigger on "quantum gate demo", "CircuitError duplicate bit arguments", "Toffoli index", "Qiskit unitary", "Aer segfault".

Working demo: `~/薦家/布球/qbit-gate-viz/app.py` — run with `streamlit run app.py`.

## Contents

- `SKILL.md` — rules, code templates, common mistakes

## License

MIT
