**Technical Test (Q3) (README)**

Bio signals/ML: A self-supervised, zero-shot signal-quality detector.

The real problem: Hand-tuned signal-quality heuristics - thresholds on autocorrelation or autoregressive stability - are quick to run but work poorly on skin tones, motion artefacts, and sensor placements. Fully supervised quality classifiers need labelled data that usually does not exist at the point of deployment.

Your task: Sketch a Python design (pseudocode is fine; a working prototype is a bonus) for a method that segments a continuous PPG recording into beats and learns, in a zero-shot, self-supervised manner, to separate physiologically plausible, periodic segments from artefact-corrupted ones - without labelled training data. You are free to build this from classical signal statistics (autocorrelation, autoregression) or propose any self-supervised representation-learning ideas (SimSiam-style contrastive learning, adversarial autoencoders etc.) - discuss and justify whichever direction you take. Sketch how you would validate the resulting quality labels against expert annotations without ever training them.

We are specifically looking for: A clear argument for why your proposed method should work, grounded in signal structure rather than architecture name-dropping, and a validation plan that could actually catch the method fooling itself.

The solution to a self-supervised, zero-shot signal-quality detector is to use a hybrid self-supervised approach where a recording-specific distribution of normal beat structure is learned and assigned a continuous quality score.

A physiologically plausible beat needs to satisfy the following constraints simultaneously:

1. It resembles neighbouring beats morphologically.
2. It has a coherent internal periodic structure.
3. The beat timing is compatible with the surrounding rhythm.
4. Its different signal representations agree with each other.
5. Small perturbations that do not affect physiology produce similar representations.

The Python Design uses the following steps:

1. Bandpass preprocessing that removes slow baseline drift and high frequency noise.
2. Beat segmentation that represents peak-to-peak beats.
3. Normalize beats onto a common phase axis with robust amplitudes.
4. Build a self-supervised "normal beat" robust statistical model.
5. Add the following independent physiological calculations:
   1. Morphology Score.
   2. Autocorrelation Structure.
   3. Spectral Concentration.
6. Train a small encoder for multi-view self-supervision with a SimSiam/BYOL-like objective.
7. Turn the embedding into a quality score by estimate the dominant physiological manifold.

Since PPG has physiological variations cannot automatically be called artefacts. every beat cannot be forced into good/bad solely, the following quality labels will be used to allow unknown states with the exact cutoff points being determined from unsupervised stability criterion:

| **Score**     | **Label**       |
| ------------- | --------------- |
| 0.75<=Q<=1.00 | "Good"          |
| 0.65<Q<0.75   | "Possibly Good" |
| 0.45<=Q<=0.65 | "Unknown"       |
| 0.25<Q<0.45   | "Possibly Bad"  |
| 0.00<=Q<=0.25 | "Bad"           |

In the event that the method fools itself, the validation plans can be implemented:

1. Synthetic corruption tests that inject known corruptions such as beat duplications and broadband noise into expert-labelled good beats to assess detector deteriorations and learn correct invariances.
2. Distribution shift tests that use evaluation stratifications such as physiological states and sensor placements by training on one recording/session while testing on another recording/session.
3. Adversarial periodic artefacts that appear superficially like pulses, but have waveforms that disagree with genuine PPG.
4. Negative control signals like shuffled beats and synthetic sinusoids that use both uses PPG morphologies and physiological continuities.

The proposed method works for the following reasons:

1. The structure appears simultaneously in waveform morphologies, pulse timings, local repetitions and autocorrelation characteristics while measurement artefacts perturb structural subsets without producing coherent alternative physiological explanations.
2. Self-supervision enables learning invariances to nuisance variations without ground-truth quality labels, so robust estimations turn consistent populations into reference manifolds.
3. Validations deliberately expose assumptions such as non-PPG signals and heavy-contamination recordings.