# Latency-Elastic Edge Inference: Predictive Thread-Level QoS on Raspberry Pi 5

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21844865.svg)](https://doi.org/10.5281/zenodo.21844865)

**Author:** Manu Nicholas Jacob  
**Email:** manunicholasjacob@gmail.com  
**ORCID:** [0009-0007-6589-6572](https://orcid.org/0009-0007-6589-6572)  
**Status:** Submitted to a journal in 2025 and not pursued further. The
measurements and the controller stand; the manuscript does not. See "On the
manuscript" below.

---

## Overview

This repository contains all code, experimental logs, and analysis notebooks for the paper:

> **Latency-Elastic Edge Inference: Predictive Thread-Level QoS on Raspberry Pi 5**  
> Manu Nicholas Jacob  
> Manuscript, 2025. Not under review.

**Key result:** A Raspberry Pi 5 running ResNet-50 inference misses 99.8% of its 80 ms latency SLOs under high Poisson load with a reactive controller. MPC-Lite, a receding-horizon thread allocator using a 5-step prediction horizon, reduces this to **0.0% violations** and cuts p99 tail latency by **3.4x** (30.0 ms vs. 102.9 ms).

---

## On the manuscript

This work was written up and submitted to a journal in 2025. It did not go
forward, and it is not currently under review anywhere. I have left the
repository up because the measurements are real and reproducible and the
controller does what the numbers below say it does, but nothing here should be
read as a paper in press.

If you want the current state of this research programme, it is at
[manunicholasjacob.com/research](https://manunicholasjacob.com/research).

---

## Repository Structure

```
latency-elastic-edge-inference/
├── controllers/
│   ├── 07_online_controller.py     # MPC-Lite, Equal Share, CPU Quota, Reactive
│   ├── 08_load_sweep.py            # Load sweep experiments
│   └── 09_ablation_runner.sh       # Ablation study runner
├── calibration/
│   ├── mu_k_calibration.ipynb      # μ(k) measurement notebook
│   └── thermal_model_fit.py        # Thermal model (α, β, γ) identification
├── experiments/
│   ├── run_all.sh                  # Master pipeline: runs all experiments
│   ├── run_main_comparison.sh      # Main 3-controller comparison
│   ├── run_ablation_prediction.sh  # Prediction horizon ablation
│   ├── run_bursty_onoff.sh         # On-Off bursty workload
│   └── run_realistic_workload.sh   # Bimodal service-time workload
├── logs/                           # All raw CSV latency logs (>110,000 requests)
│   ├── main_comparison/
│   ├── load_sweep/
│   ├── ablation_prediction/
│   ├── ablation_interval/
│   ├── ablation_freq/
│   ├── bursty_onoff/
│   ├── failure_modes/
│   ├── stability_60min/
│   ├── thermal_disturbance/
│   ├── slo_flexibility/
│   ├── realistic_workload/
│   └── 3tenant_ablation/
├── analysis/
│   ├── generate_figures.py         # Reproduces all 8 paper figures
│   ├── compute_statistics.py       # Reproduces all paper table numbers
│   └── requirements.txt            # Python dependencies
├── paper/
│   ├── paper4_tier2.tex            # Main LaTeX source
│   ├── cover_letter.tex            # Submission cover letter
│   ├── research_highlights.tex     # journal research highlights
│   └── figures/                    # All 8 paper figures (PDF)
└── README.md
```

---

## Hardware Requirements

| Component | Specification |
|-----------|--------------|
| Device | Raspberry Pi 5 (BCM2712) |
| RAM | 8 GB LPDDR4X |
| OS | Raspberry Pi OS Bookworm (64-bit), kernel 6.1 |
| Cooling | Passive heatsink only (no active fan) |
| Network | Not required (local loopback experiments) |

---

## Software Requirements

```
Python         3.11.2
NumPy          1.26.3
SciPy          1.11.4
requests       2.31.0
Flask          3.0.0
psutil         5.9.7
```

Install dependencies:
```bash
pip install -r analysis/requirements.txt
```

---

## Reproducing the Experiments

### Step 1: Calibrate μ(k) and Thermal Model (~30 minutes)
```bash
# On Raspberry Pi 5:
python calibration/mu_k_calibration.py --output logs/calibration/mu_k.json
python calibration/thermal_model_fit.py --output logs/calibration/thermal_params.json
```

### Step 2: Run All Experiments (~6-8 hours total)
```bash
# Runs all 12 experiment groups sequentially
bash experiments/run_all.sh
```

Or run individual experiments:
```bash
bash experiments/run_main_comparison.sh    # Main comparison (Table 2)
bash experiments/run_ablation_prediction.sh # Ablation (Table 3)
bash experiments/run_bursty_onoff.sh        # On-Off workload
bash experiments/run_realistic_workload.sh  # Bimodal workload (Table 4)
```

### Step 3: Generate Figures and Statistics
```bash
python analysis/generate_figures.py --logs-dir logs/ --output paper/figures/
python analysis/compute_statistics.py --logs-dir logs/
```

---

## MPC-Lite Controller

The core controller is in `controllers/07_online_controller.py`. Run modes:

```bash
# MPC-Lite (predictive, thermal-aware)
python 07_online_controller.py --mode convex --horizon 5 --interval 1.0

# Equal Share baseline
python 07_online_controller.py --mode equal

# CPU Quota baseline  
python 07_online_controller.py --mode quota

# Reactive (threshold-based) baseline
python 07_online_controller.py --mode reactive
```

---

## Key Results Summary

| Experiment | MPC-Lite | Equal Share | Reactive |
|------------|----------|-------------|----------|
| SLO violations (moderate load) | **0.0%** | 1.15% | - |
| p99 latency (moderate load) | **30.0 ms** | 102.9 ms | - |
| Goodput at high load | **13.6%** | 0.2% | 0.2% |
| Mean latency, bursty On-Off | **5.4 ms** | 17.9 ms | - |
| Long-run violation rate (60 min) | 33.2±1.0% | - | - |
| Controller CPU overhead | **0.9%** | - | - |

---

## Experimental Protocol

- **SLO target:** 80 ms  
- **Thermal cap:** 75 °C  
- **Cooldown between runs:** 60 s (until T < 50 °C)  
- **Random seed:** 42  
- **Repetitions:** 3 independent runs per condition  
- **Total requests:** >110,000  

---

## Queue Divergence Threshold

A key concept introduced in this paper: **λ\* = max_k μ(k)** is the arrival rate above which no single-device controller can maintain a bounded queue. For our Raspberry Pi 5 setup with ResNet-50 emulation:

- **λ\*** ≈ 15 req/s (4 threads, nominal temperature)
- At λ > λ\*, all controllers saturate at ~100% violations; the correct response is admission control, not thread allocation.

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
