# QUBO + QAOA for VRP (Qiskit) - Step 1

Single-vehicle VRP -> QUBO -> QAOA (Qiskit + Aer) -> compared with classical methods.
No RL yet: SciPy COBYLA tunes the QAOA angles (`QAOA.optimise` in `qaoa_qiskit.py` is what the RL agent will replace).

## Files
| File | Role |
|---|---|
| `vrp_qubo.py` | load instances (.evrp / TSPLIB-like .vrp), pick depot + k customers, build the QUBO (`arc` or `order`), convert to Ising |
| `qaoa_qiskit.py` | QAOA circuit, Aer Estimator objective, COBYLA loop, Aer sampling + exact probabilities |
| `classical_baselines.py` | brute force (exact), ILP with MTZ (exact, SciPy HiGHS), OR-tools, greedy nearest neighbour |
| `run_experiment.py` | command line runner, prints comparison tables, saves JSON |

## Install (Colab or local)
```
pip install qiskit qiskit-aer scipy numpy ortools
```
Tested with qiskit 2.5.2, qiskit-aer 0.17.2, scipy 1.17, ortools 9.15. Pin your versions; Qiskit APIs change often.

## Run
```
# Mavrovouniotis instance (download the .evrp file first), depot + 3 nearest customers
python run_experiment.py --file E-n29-k4-s7.evrp --customers 3 --kind both --p 1 2

# random instance
python run_experiment.py --customers 4 --kind order --p 1 2
```
Options: `--select nearest|first|random`, `--shots`, `--restarts`, `--maxiter`, `--seed`, `--out`.

## Notes
* Qubits: `arc` = n(n-1), `order` = (n-1)^2, where n = customers + depot. Aer statevector is comfortable up to ~20 qubits.
* `arc` only rules out every broken loop for <= 4 customers (as in the paper). More customers: broken loops are rejected at decoding.
* Penalty weights are validated once against brute force with a classical NumPy enumeration of all 2^N bit patterns
  (`QUBO.all_energies`). This is a sanity check on the formulation, not part of the quantum computation.
  It is skipped above 22 qubits.
* CMT files in VRP-REP are XML; the loader reads TSPLIB-style text files only.
* "P(optimal route)" is the exact probability (Aer `save_probabilities`) of measuring an optimal route.
  "x vs random" divides it by what uniform random bitstrings would give.
