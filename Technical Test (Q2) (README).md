**Technical Test (Q2) (README)**

Question 2 - Biosignal/Embedded: low-compute artefact rejection for battery-limited wearables.

The real problem: Signal-quality assessment is usually built on models heavy enough to noticeably shorten battery life on a wearable, yet acquiring an artefact-corrupted signal is worse than acquiring none, since it silently corrupts every biomarker computed downstream. Reducing that compute cost is one of the more direct paths to a clinical-grade device people will actually wear all day.

Your task: Describe - and implement a concept in Python - an unsupervised, feature-based method for rejecting artefact-corrupted segments of a PPG or ECG signal, cheap enough to run continuously on a low-power microcontroller without a trained ML classifier in the loop.

State which signals properties your method exploits and why, and estimate its compute cost

(operations, memory) against a typical lightweight ML alternative.

We are specifically looking for: A method you can justify from first principles about the signal rather than a black box; realistic reasoning about what "cheap enough for a battery powered wearable" means in practice; and honesty about the artefacts your approach would miss.

The solution to low-compute artefact rejection for battery-limited wearables is to use rule-based physiological signal-quality index (SQI) built from multiple physically interpretable features instead of a single classifier to detect whether the segment still has the structural properties that a cardiac waveform must have.

The physiological SQI consists of 4 features:

| **Features**             | **Detects**                             | **Meanings**                                                                                        |
| ------------------------ | --------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Flat/Clip Fractions**  | ADC Saturations/Disconnected Sensors.   | A physiological waveform cannot be constant or hit the ADC rails for a substantial window fraction. |
| **Robust Amplitudes**    | Contact Losses/Weak Signals.            | Non-zero dynamic ranges are required for cardiac pulsations.                                        |
| **Derivative Roughness** | Electrical Interference/ Motion Spikes. | More sample-to-sample variations are introduced by motion artefacts.                                |
| **Beat Periodicities.**  | Waveform Noise.                         | ECG /PPG contain repeating cardiac cycles.                                                          |

Instead of doing a Fast Fourier Transform (FFT), the candidate beats are detected using a threshold detector to examine whether the resulting beat intervals are physiologically possible and reasonably constant.

The Python Implementation uses ordinary NumPy for easy algorithm inspection in the following steps:

1. Basic preprocessing via baseline removal.
2. Feature extraction with beat detection.
3. Baseline calibration from unlabeled data.
4. Quality decision based on robust unsupervised deviation score.

The 4 features work for the following reasons:

1. Cardiac signals have constrained temporal structures, causing precise morphologies to change between people/physiologies while the existence of reasonably regular cardiac cycles is persistent, making beat periodicities a first-principles feature.
2. Motion artefacts increase local roughness, so normalizing by amplitude lower difference sensitivities in sensor gains via skin contact.
3. Sensor failures appear different from physiological variabilities, producing constant outputs, rail values, repeated sample sequences and large dynamic excursions.
4. Robust statistics avoid expensive distribution modelling systems since medians and Median Absolute Deviations (MADs) are used instead.

The Python Implementation has lower computing costs than a typical lightweight ML alternative for the following reasons:

1. The Python Implementation does not require training nor labels.
2. The Python Implementation has a higher interpretability.
3. The Python Implementation can be easily adapted to the wearer.
4. The Python Implementation has easier failure explanations.
5. The Python Implementation possess less engineering complexities.

However, the Python Implementation is missing the following artefacts:

1. The Python Implementation can accept mechanical waveforms with convincing periodicity that are produced from repetitive wrist motion at approximately the heart rate.
2. The Python Implementation can reject genuine physiologies since irregular rhythms can violate regularity assumptions.
3. Baseline drifts caused by slow motions can trick the Python Implementation.
4. Electromagnetic interferences can fool the Python Implementation.
5. Morphological corruptions can distort the Python Implementation.