# Background trial filter for active impulsive noise control

Code and aggregated results for

> Y. Sugiura, "Fast Reconvergence in Active Impulsive Noise Control Using a Background Trial Filter," submitted to *APSIPA Transactions on Signal and Information Processing*.

This repository is a snapshot of the code used for the paper. It contains the proposed algorithm, the eleven comparison algorithms, the evaluation scripts, the scripts that produce the figures and tables of the paper, and the per-seed results of the reported evaluation.

## Requirements

Python 3.11 or later with

```sh
pip install -r requirements.txt
```

The simulations run on the CPU. The full evaluation (20 seeds × 5 noise classes × 5 conditions) takes several hours with 16 processes.

## Layout

| Path | Contents |
|---|---|
| `anc.py`, `algorithms/` | Base time-domain ANC simulator and comparison algorithms |
| `atsip/explore/` | Proposed algorithm, evaluation and analysis scripts (run all commands from this folder) |
| `atsip/explore/data/` | Scripts that fetch and prepare the noise and room impulse responses, and the prepared path subset `paths16/` |
| `atsip/explore/out/` | Per-seed results of the reported evaluation (JSON, NPZ) |
| `atsip/explore/RESULTS_*.md` | Reports written by the evaluation scripts |
| `atsip/explore/paper/` | Scripts that produce the figures and tables of the paper from `out/` |
| `atsip/oracle/`, `atsip/explore/theory/` | Reference and analysis modules imported by the evaluation |

## Reproducing the paper

Regenerate the figures and tables from the stored results:

```sh
cd atsip/explore
python paper/figs.py
python paper/table_all.py
python paper/fig_diff.py
python paper/fig_stepsweep.py
python paper/anr_curves.py --common
```

Rerun the main evaluation (evaluation seeds 100–119) and write the report:

```sh
python final2.py final --procs 16
python final2.py final-report
python final2.py coef --procs 16
python final2.py coef-report
```

Other experiments of the paper:

| Experiment | Script |
|---|---|
| Comparison algorithms (C-FxLMS, MFxNLMS, dual-threshold FxLMS, APFM) | `cc.py`, `cc_class.py`, `mfx.py`, `thx.py`, `apfm.py` |
| Frozen-coefficient noise reduction | `frozen5.py` |
| Variants without trials or detector | `ablate.py` |
| Sensitivity to the detector threshold | `sens.py` |
| Secondary-path model error | `smodel.py` |
| Machine gun noise | `mgun.py` |
| Step-size sweep | `stepsweep.py` |
| Decisions and the bound of Proposition 3 | `judge.py`, `judge_extra.py`, `judge_eta.py` |
| Computational cost | `cost.py` |
| Paired seed-bootstrap intervals | `paired_ci.py` |

Each script prints its subcommands with `-h`. The settings of the proposed algorithm are fixed in `EVAL_FROZEN.json` and `TUNE*.json`. These settings were chosen without the evaluation seeds 100–119.

A quick check reruns one case and compares it with the stored result:

```python
import final2, final as F
final2.configure()
F.run_final_job(("G", "RS-far", 100, False))   # writes out/final2_run/G_RS-far_s100.json
```

## Data

The noise recordings and the raw room impulse responses are not redistributed. `data/fetch.py`, `data/make_paths.py` and `data/make_noise.py` download them from the original sources and prepare them. Sources, versions and licenses are listed in [`atsip/explore/data/README.md`](atsip/explore/data/README.md).

`atsip/explore/data/paths16/` is a processed subset of MeshRIR S32-M441 v1.2 (S. Koyama, T. Nishida, K. Kimura, T. Abe, N. Ueno, J. Brunnström, "MeshRIR: A dataset of room impulse responses on meshed grid points for evaluating sound field analysis and synthesis methods," IEEE WASPAA 2021; <https://doi.org/10.5281/zenodo.10852693>). It is redistributed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The responses were selected, resampled from 48 kHz to 16 kHz, truncated and converted to float32 (details in `paths16/paths16.json`).

## License

The code is released under the MIT License (see `LICENSE`). The MeshRIR subset in `paths16/` remains under CC BY 4.0.

## Citation

Please cite the paper and this software (see `CITATION.cff`).
