# Background trial filter for active impulsive noise control

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24589368.svg)](https://doi.org/10.5281/zenodo.24589368)

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

## Where the proposed algorithm is implemented

All paths are in `atsip/explore/`.

| Part (paper) | Code |
|---|---|
| Block loop of the frequency-domain FxNLMS | `final.py`, `simulate` |
| Spectral-factor direction (Section 4) | `final.py`, `sf_factor` and `sf_dir_cached` |
| Step size divided by max(g_n, g_min), early and late phases | `prop2.py`, `P2` and `mu_late` |
| Reconstructed error frame | `prop2.py`, `Frame`, and `final.py`, `SwitchedFrame` |
| Trial detector z_n with the guard interval | `protoC.py`, `RSNDetector`, and `det2.py`, `FreezeDetector` |
| Background trial and decisions ρ_n (discard, early transfer, final transfer) | `prop5.py`, `P5`, and `final.py`, `P5Cap` |
| Assembly of the proposed algorithm (`P5-early-cap`) | `final.py`, `run_rule`, with `final2.configure()` installing `FreezeDetector` |

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
| Variants of the proposed algorithm | `final2.py` (rules other than `P5-early-cap`) |
| Sensitivity to the detector threshold | `sens.py` |
| Secondary-path model error | `smodel.py` |
| Machine gun noise | `mgun.py` |
| Step-size sweep | `stepsweep.py` |
| Decisions and the bound of Proposition 3 | `judge.py`, `judge_extra.py`, `judge_eta.py` |
| Computational cost | `cost.py` |
| Paired seed-bootstrap intervals | `paired_ci.py` |

Each script prints its subcommands with `-h`. The settings of the proposed algorithm were chosen on the design seeds 10–49, and the step sizes of the comparison algorithms on the seeds 10–14. `EVAL_FROZEN.json` records the SHA-256 of the evaluated files. The scripts refuse the seeds 100–119 without this file.

A quick check reruns one case in a scratch folder and compares it with the stored result:

```python
import json, pathlib, tempfile
import final2, final as F
final2.configure()
tmp = pathlib.Path(tempfile.mkdtemp())
F.final_dir = lambda dry: tmp
F.run_final_job(("G", "RS-far", 100, False))
new = json.loads((tmp / "G_RS-far_s100.json").read_text())["rows"]
old = json.loads(pathlib.Path("out/final2_run/G_RS-far_s100.json").read_text())["rows"]
print(all(a[k] == b[k] for a, b in zip(new, old) for k in ("L1_whole", "rec_blocks", "div")))
```

## Labels in the code

| Code | Paper |
|---|---|
| `P5-early-cap` | Proposed |
| `nors-cap` | Without trials |
| `P5-early-noG` | Proposed without 1/g |
| `P4-cap` | Immediate restart |
| `TP-always-cap` | Always-on background trial |
| `ND-always-cap` | Always reconstructed error |
| `SF-fix` | Fixed-step SF |
| `Freq_FxNLMS_CohVSS` | CohVSS-FxNLMS |
| `G`, `II300`, `a145`, `a120` | Gaussian, Gaussian with isolated impulses, α-stable (α = 1.45, 1.20) |
| `stat`, `RS-near`, `RS-far`, `EN0`, `ENm10` | No change, close source, far source, disturbance at 0 dB and −10 dB |

## Data

The noise recordings and the raw room impulse responses are not redistributed. `data/fetch.py`, `data/make_paths.py` and `data/make_noise.py` download them from the original sources and prepare them. Sources, versions and licenses are listed in [`atsip/explore/data/README.md`](atsip/explore/data/README.md).

`atsip/explore/data/paths16/` is a processed subset of MeshRIR S32-M441 v1.2 (S. Koyama, T. Nishida, K. Kimura, T. Abe, N. Ueno, J. Brunnström, "MeshRIR: A dataset of room impulse responses on meshed grid points for evaluating sound field analysis and synthesis methods," IEEE WASPAA 2021; <https://doi.org/10.5281/zenodo.10852693>). It is redistributed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The responses were selected, resampled from 48 kHz to 16 kHz, truncated and converted to float32 (details in `paths16/paths16.json`).

## License

The code is released under the MIT License (see `LICENSE`). The MeshRIR subset in `paths16/` remains under CC BY 4.0.

## Citation

Please cite the paper and this software, <https://doi.org/10.5281/zenodo.24589368> (see `CITATION.cff`).
