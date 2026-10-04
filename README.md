# Finding Low-Rank Matrix Multiplication Tensors

A research prototype for finding bilinear algorithms for matrix multiplication using SAT and SMT solvers.

The project encodes rank-\(r\) decompositions of the matrix multiplication tensor \(\langle n,m,p\rangle\), experiments with symmetry-breaking constraints and valid inequalities, and compares solver behavior on small instances.

## Mathematical Formulation

For matrices \(A\in F^{n\times m}\) and \(B\in F^{m\times p}\), matrix multiplication can be expressed as a bilinear decomposition

$$
AB=\sum_{\ell=1}^{r} A_\ell(A)B_\ell(B)C_\ell,
$$

where the linear forms and output matrices define a rank-\(r\) matrix multiplication scheme.

Equivalently, the matrix multiplication tensor admits a decomposition

$$
\langle n,m,p\rangle
=
\sum_{\ell=1}^{r} a_\ell\otimes b_\ell\otimes c_\ell.
$$

The goal is to find such decompositions for small dimensions and investigate the computational difficulty of proving lower bounds.

This repository includes:

* Boolean encodings over \(\mathbb{F}_2\).
* Integer encodings over finite coefficient sets, including \(\{-1,0,1\}\).
* Symmetry-breaking constraints and valid inequalities.
* CNF generation for external SAT solvers.
* Recorded solver experiments and candidate witness schemes.

The integer encodings use specified coefficient domains; they do not constitute an unrestricted search over all integer coefficients.

## Repository Structure

| Path                              | Description                                                      |
| --------------------------------- | ---------------------------------------------------------------- |
| `scripts/z3_solve.py`             | Unified Z3 driver with Boolean and integer encoding modes        |
| `scripts/constraint_programming/` | Integer constraint formulations                                  |
| `scripts/sat_solving/`            | Boolean encoding, CNF conversion, and external SAT pipeline      |
| `scripts/mm_scheme.py`            | `MM_Scheme` representation; work in progress                     |
| `results/`                        | Recorded solver outputs, timings, logs, and candidate schemes    |
| `archive/`                        | Superseded implementations and historical experimental artifacts |
| `tests/`                          | Current test scripts                                             |
| `todo.txt`                        | Known issues and future work                                     |

## Setup

Requires Python 3 and the dependencies listed in `requirements.txt`.

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

The external SAT pipeline additionally requires a compatible solver, such as [Kissat](https://github.com/arminbiere/kissat).

## Running the Experiments

### Z3: Boolean or integer encoding

The unified driver takes the matrix dimensions and target rank as command-line arguments:

```sh
python3 scripts/z3_solve.py 2 2 2 7
```

The program interactively requests the encoding mode, coefficient domain, and optional pruning constraints.

* `sat`: Boolean encoding over \(\mathbb{F}_2\) when using coefficients \(\{0,1\}\).
* `smt`: Integer encoding over the selected finite coefficient set.

**Important:** Boolean variables with coefficients \(\{-1,0,1\}\) do not represent the same problem as a decomposition over \(\mathbb{F}_2\).

### Integer constraint programming

The \(\{-1,0,1\}\) formulation is available at:

```sh
python3 scripts/constraint_programming/cp_minus1_0_1.py
```

It prompts for the tensor dimensions and target rank.

### External SAT pipeline

From `scripts/sat_solving/`:

```sh
python3 get_unformatted_cnf.py
python3 unformatted_to_dimacs.py unformatted_cnf_files/rank_222_leq_7_unformatted.txt
kissat cnf_files/rank_222_leq_7.cnf
```

The scripts prompt for the dimensions and rank during generation.

Some scripts write logs and intermediate artifacts relative to the current working directory. Run them from their expected directories.

## Experimental Results

The following results are transcribed from the recorded experiments in `results/`. They are historical observations, not independently certified results.

### Z3 and constraint-programming experiments

| Tensor                  | Encoding               | Coefficients   | Rank | Result  | Recorded time |
| ----------------------- | ---------------------- | -------------- | ---: | ------- | ------------: |
| \(\langle2,2,2\rangle\) | Boolean                | \(\{0,1\}\)    |    7 | SAT     |        0.35 s |
| \(\langle2,2,2\rangle\) | Boolean                | \(\{-1,0,1\}\) |    6 | Unknown |        1.18 s |
| \(\langle2,2,2\rangle\) | Integer SMT            | \(\{-1,0,1\}\) |    7 | SAT     |      110.90 s |
| \(\langle2,2,3\rangle\) | Boolean                | \(\{0,1\}\)    |   11 | SAT     |        8.26 s |
| \(\langle2,2,3\rangle\) | Constraint programming | \(\{-1,0,1\}\) |   11 | Unknown |      424205 s |
| \(\langle2,3,3\rangle\) | Boolean                | \(\{0,1\}\)    |   15 | Unknown |      612008 s |

### External SAT solver timings

Recorded in `results/sat_timings.txt`. Outcomes were not preserved for these runs.

| Tensor                  | Rank |   Kissat |     YalSAT |
| ----------------------- | ---: | -------: | ---------: |
| \(\langle2,2,2\rangle\) |    7 |   0.18 s |     3.83 s |
| \(\langle2,2,2\rangle\) |    6 | 122.67 s |          — |
| \(\langle2,2,3\rangle\) |   11 |   3.41 s | 24881.44 s |

Candidate witness schemes are preserved in:

* `results/solutions/2_2_2_rank7.txt`
* `results/solutions/2_2_3_rank11.txt`

Raw solver traces are in `results/logs/`.

## Correctness and Limitations

The encodings and pruning constraints are experimental and have not yet been independently validated in all cases.

Some recorded instances produce differing outcomes across encoding modes. The cause has not been fully isolated; possible issues include the encoding itself and symmetry-breaking constraints.

Interpret results carefully:

* `sat` indicates that the encoded constraints have a satisfying assignment. A candidate decomposition should still be checked against the tensor equations.
* `unsat` establishes nonexistence only for the exact encoded problem, including its coefficient domain and all added constraints.
* `unknown` is inconclusive and does not establish either existence or nonexistence.
* Solver timings are historical measurements and may depend heavily on solver version, machine, and configuration.

The project is intended as a research and experimentation platform, not a certified matrix multiplication rank prover.

## Future Work

Current priorities include:

* Independently verifying candidate decomposition witnesses.
* Auditing symmetry-breaking constraints for soundness.
* Improving reproducibility of experiments.
* Separating reusable encoding logic from interactive drivers.
* Expanding automated tests for small tensor instances.

See `todo.txt` for additional development notes.

