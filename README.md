# Latency-Elastic Edge Inference: Predictive Thread-Level QoS on Raspberry Pi 5

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21844865.svg)](https://doi.org/10.5281/zenodo.21844865)

**Author:** Manu Nicholas Jacob  
**Email:** manunicholasjacob@gmail.com  
**ORCID:** [0009-0007-6589-6572](https://orcid.org/0009-0007-6589-6572)  
**Status:** Submitted to a journal in 2025 and not pursued further. The
manuscript is not under review. See "On the manuscript" below.

---

## Overview

This repository holds the aggregated measurements, figures and LaTeX source for:

> **Latency-Elastic Edge Inference: Predictive Thread-Level QoS on Raspberry Pi 5**  
> Manu Nicholas Jacob  
> Manuscript, 2025. Not under review.

The study compares three runtime controllers (Reactive, PI-SLO and MPC-Lite) that
adjust batch size and thread count for INT8 ONNX Runtime inference of
MobileNetV3-Small and ResNet-18 (CIFAR-100) on a Raspberry Pi 5, under Poisson,
Pareto and On-Off arrivals at offered loads rho = 0.6, 0.75, 0.85 and 0.95.

**Key result (from `policy_table.csv`):** for ResNet-18 under Poisson arrivals at
rho = 0.95, MPC-Lite holds p99 latency at 8.16 ms against 11.54 ms for Reactive and
11.50 ms for PI-SLO (medians across runs). No controller violates its SLO (12 ms for
MobileNetV3-Small, 20 ms for ResNet-18) in any committed scenario; the largest
per-run violation rate in `scenario_table.csv` is 0.0013%.

---

## On the manuscript

This work was written up and submitted to a journal in 2025. It did not go
forward, and it is not currently under review anywhere. The tables and figures
here are the measurements behind it; nothing here should be read as a paper in
press.

If you want the current state of this research programme, it is at
[manunicholasjacob.com/research](https://manunicholasjacob.com/research).

---

## What is in this repository

| file | contents |
|---|---|
| `scenario_table.csv` | one row per run: model, controller, arrival, rho, run id, p50/p90/p99 latency (ms), mean latency, mean queue length, SLO (ms), SLO violation %, sample count (278 runs, 11,682,447 requests) |
| `policy_table.csv` | per model x controller x arrival x rho: medians across runs of the per-run statistics (72 rows) |
| `controller_rank_highrho.csv` | controller ranking by median p90 at high load |
| `figs/` | the manuscript figures (latency vs load, SLO-violation heatmaps, queue length, topology, block diagram) |
| `main.tex`, `refs.bib` | manuscript source |
| `requirements.txt` | Python packages used by the 2025 analysis |

The controller and orchestration scripts and the raw per-request latency logs
are not part of this repository.

### Summary of the committed data

p99 latency in ms, median across runs, range over the 12 arrival x rho scenarios:

| model | Reactive | PI-SLO | MPC-Lite |
|---|---|---|---|
| MobileNetV3-Small (SLO 12 ms) | 2.19 to 3.43 | 2.11 to 3.48 | 2.09 to 2.31 |
| ResNet-18 (SLO 20 ms) | 11.01 to 11.82 | 11.00 to 12.02 | 8.13 to 9.51 |

### Correction (October 2026)

Earlier versions of this README described a different package: controller,
calibration and analysis scripts, a `logs/` tree, a ResNet-50 workload with an
80 ms SLO, Equal Share and CPU Quota baselines, and a headline of 0.0% SLO
violations with a 3.4x p99 reduction (30.0 ms vs 102.9 ms). None of those files
or numbers are in this repository, so the claims could not be checked against it
and have been removed. The text above describes only what is committed.

---

## Citation

If you use this code or data in your research, please cite:

```bibtex
@software{jacob_latency_elastic,
  author  = {Jacob, Manu Nicholas},
  title   = {Latency-Elastic Edge Inference: Predictive Thread-Level QoS on
             Raspberry Pi 5},
  year    = {2026},
  doi     = {10.5281/zenodo.21844865},
  url     = {https://github.com/manunicholasjacob/latency-elastic-edge-inference}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Contact

**Manu Nicholas Jacob**  
manunicholasjacob@gmail.com  
ORCID: [0009-0007-6589-6572](https://orcid.org/0009-0007-6589-6572)

## Archived version

This artifact is archived on Zenodo. The concept DOI
[10.5281/zenodo.21844865](https://doi.org/10.5281/zenodo.21844865)
always resolves to the latest release, and `CITATION.cff` carries the full metadata,
which is what GitHub's "Cite this repository" button renders.
