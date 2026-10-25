# iEEG-HMM

Tutorials for modelling intracranial EEG (iEEG) with a time-delay embedded
hidden Markov model (TDE-HMM): train a model on your own recordings, choose and
label its states, and apply it to describe the transient spectral states in a
recording. A pretrained model from the HUP epilepsy cohort is included.

The model is a **single-channel** TDE-HMM trained with
[osl-dynamics](https://github.com/OHBA-analysis/osl-dynamics). Each channel is
modelled independently, so one model applies to any number of channels from any
electrode layout (depth electrodes, strips, grids). The states are zero-mean
Gaussians over the time-delay embedded signal, so they differ in their power
spectrum; the model segments every channel into a sequence of recurring,
spectrally distinct states lasting of the order of 100 ms.

## Contents

| Path | Description |
|---|---|
| `1_preprocess.ipynb` | Download six example recordings and preprocess them (bipolar reference, notch and band-pass filters, resampling) |
| `2_train.ipynb` | Train the TDE-HMM from several seeds, check they agree (held-out free energy, state agreement), label the states in a fixed order and save the chosen model |
| `3_apply.ipynb` | Apply a model to a recording: state probabilities, state time courses, state spectra, summary statistics; compare the trained and released models |
| `hup-model/` | Pretrained 4-state model from the HUP cohort: `config.yml`, `model.weights.h5`, TDE-PCA projection (`pca_components.npy`), state covariances and transition matrix, and `model_info.json` (settings, training data, model selection) |

The notebooks are stored with their outputs, so they can be read without
running them.

## Getting started

The notebooks run in an [osl-dynamics](https://osl-dynamics.readthedocs.io)
environment (which brings in TensorFlow, MNE, NumPy, SciPy, pandas and
matplotlib), plus JupyterLab. Both are on conda-forge:

```bash
conda create -n osld -c conda-forge python=3.12 osl-dynamics jupyterlab
conda activate osld

git clone https://github.com/bushlab-ucl/iEEG-HMM.git
cd iEEG-HMM
jupyter lab
```

The conda package installs a GPU-enabled TensorFlow when a CUDA GPU is
available; see the osl-dynamics
[installation guide](https://osl-dynamics.readthedocs.io/en/latest/install.html)
for platform-specific notes. The notebooks were run with osl-dynamics 3.8 and
Python 3.12.

Run the notebooks in order. `1_preprocess.ipynb` downloads six 5-minute
interictal stereo-EEG recordings (about 120 MB) from
[OpenNeuro ds004100](https://openneuro.org/datasets/ds004100). `2_train.ipynb`
trains the 4-state model from three seeds (20 epochs each); on a laptop CPU
each run takes about two to three minutes. `3_apply.ipynb` runs in
under a minute with the released model, so it can also be run on its own after
`1_preprocess.ipynb`.

## Preprocessing

Data must be preprocessed as the training data were:

| Step | Setting |
|---|---|
| Channels | good iEEG contacts only |
| Reference | bipolar (adjacent contacts on the same shaft, strip or grid row) |
| Notch filter | line frequency and harmonics |
| Band-pass filter | 0.1-45 Hz, FIR (zero-phase) |
| Sampling frequency | 100 Hz |
| Standardisation | each channel z-scored, done inside the model's data preparation |

## Model settings

| Setting | Value |
|---|---|
| Time-delay embedding | 11 samples (110 ms) |
| PCA components | 6 |
| States | 4 |
| Observation model | zero-mean Gaussian, state-specific covariance |
| Sequence length, batch size | 200 samples, 256 sequences |
| Learning rate, epochs | 0.01, 20 |

After time-delay embedding and PCA, each channel's components are standardised
on their own, so the states describe transient departures from a channel's own
average spectrum rather than differences between channels.

## State labels

The order of HMM states after training is arbitrary, so states are labelled by
a fixed rule computed from their spectra on held-out subjects: **State 1** is
the state with the most 13-45 Hz power (the broadband state, which in epilepsy
recordings captures interictal discharges), and the remaining states follow in
descending 1-4 Hz power. The saved models' states are stored in this order.

## The pretrained model

- **Data:** interictal stereo-EEG (36 subjects) and electrocorticography
  (20 subjects) from the HUP epilepsy cohort, 25 randomly selected bipolar
  channels per subject.
- **Split:** 45 subjects for training (29 SEEG + 16 ECoG, about 188
  channel-hours), 11 held out (7 SEEG + 4 ECoG, about 44 channel-hours).
- **Selection:** 3 to 6 states were each trained from three seeds. Four states
  was the largest number for which all seeds converged to the same solution
  (more than 99% agreement in the most probable state on the held-out
  subjects); the released model is the four-state run with the lowest held-out
  free energy.
- **Across modalities:** models trained on SEEG only or ECoG only find the same
  four states (95-97% agreement with the released model on held-out subjects of
  either modality), so the states are not specific to one recording type.

Full details are in `hup-model/model_info.json`.
