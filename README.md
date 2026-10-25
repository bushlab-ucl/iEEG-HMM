# iEEG-HMM

A pretrained hidden Markov model (HMM) for intracranial EEG, with a tutorial
showing how to apply it to your own recordings.

The model is a single-channel time-delay-embedded HMM (TDE-HMM) with 4 states,
trained with [osl-dynamics](https://github.com/OHBA-analysis/osl-dynamics). Each
channel is modelled independently, so it can be applied to any number of
channels from any electrode layout. The states differ in their spectral content
and are numbered State 1-4 in order of decreasing low-frequency (1-4 Hz) power.

## Contents

| Path | Description |
|---|---|
| `tutorial.ipynb` | Tutorial: preprocess a recording, infer state probabilities, plot state time courses and spectra, compute summary statistics |
| `model/` | Pretrained model (`config.yml`, `model.weights.h5`), TDE-PCA projection (`pca_components.npy`) and model details (`model_info.json`); states are stored in State 1-4 order |
| `environment.yml` | Conda environment with everything needed to run the tutorial |

## Getting started

```bash
git clone https://github.com/bushlab-ucl/iEEG-HMM.git
cd iEEG-HMM
conda env create -f environment.yml
conda activate ieeg-hmm
jupyter lab tutorial.ipynb
```

The tutorial downloads a 5-minute example recording (~16 MB) from OpenNeuro.

## Preprocessing

Data must be preprocessed as the training data were:

| Step | Setting |
|---|---|
| Channels | good iEEG contacts only |
| Reference | bipolar (adjacent contacts on the same shaft, strip or grid row) |
| Notch filter | line frequency and harmonics |
| Band-pass filter | 0.1-45 Hz, FIR (zero-phase) |
| Sampling frequency | 100 Hz |
| Standardisation | each channel z-scored (done by osl-dynamics `tde_pca`) |

No bad-segment removal is needed.

## Training data and model selection

- **Data:** interictal stereo-EEG (SEEG, 36 subjects) and electrocorticography
  (ECoG, 20 subjects) from the HUP epilepsy cohort
  ([OpenNeuro ds004100](https://openneuro.org/datasets/ds004100)),
  25 randomly selected bipolar channels per subject.
- **Split:** 45 subjects for training (29 SEEG + 16 ECoG, ~188 channel-hours)
  and 11 held-out subjects for validation (7 SEEG + 4 ECoG, ~44 channel-hours).
- **Model:** 11-sample time-delay embedding (110 ms) reduced to 6 principal
  components; states are zero-mean Gaussians that differ in covariance.
- **Selection:** 3, 4 and 5 states were each trained from 3 random seeds. With
  4 states, all seeds converged to the same solution (>99% agreement in the
  most probable state on the held-out subjects); the released model is the run
  with the lowest free energy on the held-out subjects.
- **Across modalities:** HMMs trained on SEEG only or ECoG only find the same
  four states as the released model (95-97% agreement in the most probable
  state on held-out subjects of either modality; 92-94% between the SEEG-only
  and ECoG-only models), so the states are not specific to one recording type.

Full details are in `model/model_info.json`.
