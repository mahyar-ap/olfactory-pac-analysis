# Olfactory EEG Phase-Amplitude Coupling Analysis

EEG-based theta-gamma coupling analysis during olfactory stimulation across Healthy, MCI, and Alzheimer''s Disease subjects.

---

## Overview

This project investigates **Phase-Amplitude Coupling (PAC)** in EEG signals recorded during olfactory stimulation to distinguish between three cognitive states:

| Group | Label |
|---|---|
| Healthy Controls | HC |
| Mild Cognitive Impairment | MCI |
| Alzheimer''s Disease | AD |

The analysis is divided into two phases:
- **Phase 1** — Preprocessing and exploratory analysis using **MATLAB + EEGLAB**
- **Phase 2** — PAC computation and comparison using **Python (MNE-Python)**

The core scientific question: *Does theta-gamma coupling in frontal/parietal regions differ between cognitively healthy subjects and those with MCI or AD during an olfactory task?*

---

## Background

**Phase-Amplitude Coupling (PAC)** is a cross-frequency interaction where the amplitude of high-frequency neural oscillations (gamma: 30-50 Hz) is modulated by the phase of a slower oscillation (theta: 4-8 Hz). This mechanism is thought to underlie working memory and sensory encoding.

In Alzheimer''s research, theta-gamma PAC has been shown to follow a gradient:

```
Healthy (HC) > Mild Cognitive Impairment (MCI) > Alzheimer's Disease (AD)
```

This project tests this hypothesis using real EEG recordings during **chocolate** (Odor 5) and **rose** (Odor 6) olfactory stimulation.

---

## Project Structure

```
olfactory-pac-analysis/
|
|-- olfactory_pac_analysis.ipynb     # Main Python analysis notebook (Phase 2)
|-- Phase1_Report.pdf                # Phase 1 MATLAB analysis report
|-- Phase1_Description.pdf          # Assignment specification - Phase 1
|-- Phase2_Description.pdf          # Assignment specification - Phase 2
|
|-- EEG_data/                        # Processed EEG datasets (.mat)
|   |-- Healthy_control.mat
|   |-- MCI_control.mat
|   |-- Mild_control.mat
|   |-- Healthy_datasets/
|   |-- MCI_control_datasets/
|   `-- MildControl_datasets/
|
`-- PAC_Results/                     # Saved PAC computation outputs (.npz)
    |-- MVL_Results/                 # Mean Vector Length results
    |   |-- Subject_Healthy/
    |   |-- Subject_MCI/
    |   `-- Subject_AD/
    `-- Mi results/                  # Modulation Index results
        |-- SubjectHealthy/
        |-- SubjectMCI/
        `-- SubjectAD/
```

---

## Methods

### Phase 1 — MATLAB + EEGLAB

- Loaded raw EEG `.set` files using EEGLAB
- Performed **epoch extraction** around odor onset events (tmin = -2 s, tmax = +5 s)
- Applied preprocessing: filtering, artifact inspection
- Separated epochs by stimulus type (Odor 5 / Odor 6) and group (HC / MCI / AD)

### Phase 2 — Python (MNE-Python)

#### 1. Mean Vector Length (MVL)

MVL uses the **Continuous Wavelet Transform (CWT)** with Morlet wavelets for time-frequency decomposition — no bandpass filtering required.

The complex PAC vector is formed as:

```
z(t) = A_gamma(t) * exp(j * phi_theta(t))
```

MVL is computed over a **sliding window** (1 s width, 95% overlap), then averaged across time, frequencies, and trials.

#### 2. Modulation Index (MI)

MI uses **bandpass filtering + Hilbert transform** to extract:
- Theta phase from the 4-8 Hz band
- Gamma amplitude from the 30-50 Hz band

Gamma amplitude is binned into **18 phase bins** (-pi to pi), normalized to a probability distribution P, then compared to a uniform distribution Q via **Kullback-Leibler divergence**:

```
MI = D_KL(P || Q) / log(N)
```

A **trial rejection** step (z-score + absolute amplitude threshold) removes noisy epochs before MI computation.

#### Channel Pairs Analyzed

| Pair | Description |
|---|---|
| Fz to Fz | Local frontal PAC |
| Pz to Pz | Local parietal PAC |
| Fz to Pz | Rostro-caudal (long-range) |
| Pz to Fz | Caudo-rostral (long-range) |
| FP1 to FP1 | Prefrontal PAC |

---

## Key Findings

### MVL Results
- Stimulus-locked PAC increases are most prominent in **AD at frontal sites**, suggesting compensatory hyperactivity
- Long-range coupling (Fz-Pz) differs notably between groups
- MVL values are on the order of 1e-7; relative differences between groups are the interpretive focus

### Modulation Index Results

| Subject | Pair | Stimulus | MI | Observation |
|---|---|---|---|---|
| Healthy | Pz to Fz | Chocolate | 0.00071 | Focal, organized coupling |
| Healthy | Pz to Fz | Rose | 0.00084 | Clear stimulus-specific shift |
| Healthy | Fz to Pz | Chocolate | 0.00113 | Strong phase specificity (~300-330 deg) |
| MCI | Fz to Pz | Chocolate | 0.00110 | Broader, less focal distribution |
| AD | — | — | — | Most diffuse; lowest phase specificity |

**Pattern observed:**
- **Healthy** — compact, stimulus-specific PAC with clear preferred phases
- **MCI** — broader distributions, partial coupling maintained
- **AD** — most diffuse histograms, degraded phase-locking

### Comparison: MVL vs. MI

| Feature | MVL | MI |
|---|---|---|
| Sensitivity to coupling | Higher | Lower |
| Noise robustness | Lower | Higher |
| Method | CWT + sliding window | Bandpass + Hilbert + KL divergence |
| Computational cost | Higher | Lower |

---

## Dependencies

### Python (Phase 2)

```
mne
numpy
scipy
matplotlib
```

Install with:

```bash
pip install mne numpy scipy matplotlib
```

### MATLAB (Phase 1)
- MATLAB R2022+
- [EEGLAB](https://sccn.ucsd.edu/eeglab/) toolbox

---

## Usage

### Running Phase 2 Analysis (Python)

1. Ensure `Healthy.set`, `MCI.set`, and `MAD.set` are in the project root
2. Open and run `olfactory_pac_analysis.ipynb` in Jupyter
3. Results (`.npz` files) will be saved to `PAC_Results/`

To reproduce a specific channel-pair plot:

```python
# MVL analysis
Plot_MVL(''Fz'', ''Pz'')

# MI analysis
Plot_MI(''Fz'', ''Pz'')
```

---

## References

- Tort, A. B. et al. (2010). *Measuring phase-amplitude coupling between neuronal oscillations of different frequencies.* J. Neurophysiol.
- Canolty, R. T. & Knight, R. T. (2010). *The functional role of cross-frequency coupling.* Trends Cogn. Sci.
- PMC article on PAC reduction in MCI/AD: [PMC9396580](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9396580/)
