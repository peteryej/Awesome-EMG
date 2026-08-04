# EMG Modeling Methods

This document tracks recent modeling approaches for electromyography (EMG), with an emphasis on surface EMG (sEMG) methods published in 2024 or later. It is intended as a living reference rather than an exhaustive review. Peer-reviewed papers and clearly labeled preprints are both included when they introduce a useful modeling direction. Last updated: August 2026.

## Contents

- [Preprocessing](#preprocessing)
- [Gesture recognition](#gesture-recognition)
- [Pose estimation](#pose-estimation)
- [Fusion (EMG + IMU, EMG + CV)](#fusion-emg--imu-emg--cv)
- [Force estimation](#force-estimation)

## Preprocessing

Recent learned preprocessing methods increasingly replace contaminant-specific filters with data-driven denoisers that model both local waveform structure and longer temporal context.

| Paper | Year / venue | Modeling focus | Resources |
|---|---|---|---|
| TrustEMG-Net: Using Representation-Masking Transformer with U-Net for Surface Electromyography Enhancement | 2024, IEEE Journal of Biomedical and Health Informatics (online publication) | General-purpose sEMG denoising with a U-Net/Transformer denoising autoencoder | [Paper](https://doi.org/10.1109/JBHI.2024.3475817) · [Code](https://github.com/eric-wang135/TrustEMG) |
| SDEMG: Score-Based Diffusion Model for Surface Electromyographic Signal Denoising | 2024, IEEE ICASSP | Score-based diffusion denoising of ECG-contaminated sEMG | [Paper](https://arxiv.org/abs/2402.03808) · [Code](https://github.com/tonyliu0910/SDEMG) |

### TrustEMG-Net

Wang et al. formulate sEMG enhancement as denoising autoencoding and combine U-Net's multiscale local features with a Transformer encoder and representation masking. They evaluate on NinaPro signals corrupted by five common contaminant types across input SNRs from -14 to 2 dB. The reported results improve over the compared denoisers on five signal-quality metrics, with at least a 20% gain reported in the paper, suggesting that one learned model can cover a wider noise range than a contaminant-specific heuristic. The evaluation uses synthetically mixed contamination, so performance on naturally occurring, nonstationary artifacts should still be validated for each deployment.

### SDEMG

Liu et al. apply a score-based diffusion model to reconstruct clean sEMG from signals contaminated by electrocardiogram activity. Training and evaluation mix sEMG from the Non-Invasive Adaptive Prosthetics database with ECG from the MIT-BIH Normal Sinus Rhythm Database, and the model outperforms the paper's comparison methods on the resulting benchmark. The work is especially relevant for recordings near the trunk, where cardiac interference is prominent, but its narrower ECG-removal objective does not by itself address motion artifacts, power-line noise, electrode shift, or broadband sensor noise.

## Gesture recognition

Current work focuses on generic cross-user decoding, robustness to session and posture shifts, efficient inference, and lightweight personalization.

| Paper | Year / venue | Modeling focus | Resources |
|---|---|---|---|
| A Generic Non-Invasive Neuromotor Interface for Human-Computer Interaction | 2025, Nature | Large-scale generic sEMG models for gestures, continuous control, and handwriting | [Paper](https://doi.org/10.1038/s41586-025-09255-w) · [Code and data](https://github.com/facebookresearch/generic-neuromotor-interface) |
| SpGesture: Source-Free Domain-Adaptive sEMG-Based Gesture Recognition with Jaccard Attentive Spiking Neural Network | 2024, NeurIPS | Low-latency spiking recognition with source-free adaptation to distribution shift | [Paper](https://proceedings.neurips.cc/paper_files/paper/2024/hash/409334f42cbb57d07aa152f2d0433ec7-Abstract-Conference.html) · [Code](https://github.com/guoweiyu/SpGesture) |

### A Generic Non-Invasive Neuromotor Interface for Human-Computer Interaction

Kaifosh, Reardon, and CTRL-labs at Reality Labs combine a sensitive wrist-worn sEMG device, data from thousands of consenting participants, and large generic decoding models designed to work out of the box for new users. Closed-loop evaluations cover continuous navigation, discrete gesture detection, and handwriting; the paper reports median rates of 0.66 target acquisitions per second, 0.88 gesture detections per second, and 20.9 words per minute, while personalization improves handwriting decoding by 16%. The central modeling lesson is that dataset and participant scale can turn strongly user-dependent sEMG into a generic interface, while targeted calibration remains useful for the highest-bandwidth tasks.

### SpGesture

Guo et al. introduce a spiking neural network that combines Spiking Jaccard Attention for discriminative sEMG features with a source-free domain-adaptation method that uses a pretrained source model and unlabeled target data without retaining the source dataset. On a dataset designed around different forearm postures, SpGesture reaches 89.26% accuracy and runs with reported CPU latency below 100 ms. This makes the paper useful for power- and latency-constrained wearables, while its target-time adaptation assumption means a deployment still needs a safe way to collect and curate unlabeled target-user samples.

## Pose estimation

Unlike gesture classification, pose estimation maps EMG to continuous joint configurations or trajectories and must handle ambiguity, temporal history, electrode placement, and inter-user anatomy.

| Paper | Year / venue | Modeling focus | Resources |
|---|---|---|---|
| emg2pose: A Large and Diverse Benchmark for Surface Electromyographic Hand Pose Estimation | 2024, NeurIPS Datasets and Benchmarks | Large-scale continuous hand-pose benchmark with regression and tracking baselines | [Paper](https://proceedings.neurips.cc/paper_files/paper/2024/hash/64f8884f6ba3d9ace5bb647c4f917896-Abstract-Datasets_and_Benchmarks_Track.html) · [Code and data](https://github.com/facebookresearch/emg2pose) |
| REACT: A Conditioning Framework for User-Adaptive sEMG Hand Pose Estimation | 2026, arXiv preprint | Few-shot user conditioning of a frozen EMG-to-pose model with FiLM | [Paper](https://arxiv.org/abs/2605.30127) |

### emg2pose

Salter et al. release synchronized 16-channel, 2 kHz wrist sEMG and motion-capture pose labels for 193 users, 370 hours, and 29 movement stages. The benchmark defines continuous pose regression and pose tracking, provides three recurrent baselines, and separates held-out-user, held-out-stage, and combined user-stage tests so that generalization failures cannot be hidden by random window splits. Beyond the baseline models, the paper's main contribution is a public scale and evaluation protocol comparable to modern vision datasets, making it a strong default benchmark for EMG-to-pose research.

### REACT

Xie and Cheung address unseen-user degradation by freezing a pretrained EMG-to-pose backbone, encoding a small calibration set into a compact user embedding, and applying feature-wise linear modulation (FiLM) to condition the shared representation. On emg2pose, the preprint reports improvements across all three generalization splits for both regression and tracking, including angular-error reductions of up to 3.9%, with under 45 seconds of per-user calibration and no gradient updates at deployment. REACT is a useful lightweight personalization direction, but its results should be treated as preliminary until the 2026 preprint receives independent or peer-reviewed validation.

## Fusion (EMG + IMU, EMG + CV)

Fusion methods combine EMG's direct view of muscle activation with complementary motion or scene context. IMUs are especially useful for dynamic motion and limb orientation, while cameras can contribute object affordances and grasp context.

| Paper | Modalities | Year / venue | Modeling focus | Resources |
|---|---|---|---|---|
| Discrete Gesture Recognition Using Multimodal PPG, IMU, and Single-Channel EMG Recorded at the Wrist | EMG + accelerometer/IMU + PPG | 2024, IEEE Sensors Letters | Compact wrist gesture recognition with feature-level multimodal fusion | [Paper](https://doi.org/10.1109/LSENS.2024.3447240) |
| Multimodal Fusion of EMG and Vision for Human Grasp Intent Inference in Prosthetic Hand Control | EMG + egocentric vision + eye gaze | 2024, Frontiers in Robotics and AI | Bayesian evidence fusion for early grasp-intent classification | [Paper](https://doi.org/10.3389/frobt.2024.1312554) |

### Discrete Gesture Recognition Using Multimodal PPG, IMU, and Single-Channel EMG Recorded at the Wrist

Eddy et al. study a watch-like device containing one EMG channel, a three-axis accelerometer, and photoplethysmography, using the complementary signals to reduce dependence on a spatially distributed EMG array. Multimodal fusion significantly improves recognition over each individual sensor and is more robust in cross-day and unseen-limb-position tests. The result supports compact EMG+IMU wearables, but because PPG is included in the best multimodal configuration, follow-up experiments should isolate how much of the gain comes specifically from inertial information versus vascular or motion information in PPG.

### Multimodal Fusion of EMG and Vision for Human Grasp Intent Inference in Prosthetic Hand Control

Zandigohar et al. combine a dynamic forearm-EMG classifier with an egocentric object detector and eye-gaze selection, then fuse their class-probability evidence using a Bayesian graphical model. On synchronized recordings from five participants, fusion reaches 95.3% grasp-classification accuracy during the reaching phase, compared with 81.64% for EMG and 80.5% for vision alone, showing how the modalities compensate for muscle ambiguity and visual occlusion at different phases of a grasp. The small cohort and offline evaluation limit claims of broad generalization, but the phase-by-phase analysis is a useful design template for prosthetic controllers that must infer intent before contact.

## Force estimation

Force models map EMG to continuous kinetics such as endpoint force, grip force, or spatial pressure. Recent work combines learned representations with pose context, while classical regressors remain competitive for small datasets.

| Paper | Year / venue | Modeling focus | Resources |
|---|---|---|---|
| Posture-Informed Muscular Force Learning for Robust Hand Pressure Estimation | 2024, NeurIPS | Whole-hand pressure estimation from sEMG conditioned on 3D hand posture | [Paper](https://openreview.net/forum?id=LtS7pP8rEn) · [Code and data](https://github.com/HCI-Tech-Lab/PiMForce) |
| Force Estimation for Human-Robot Interaction Using Electromyogram Signals from Varied Arm Postures | 2024, EURASIP Journal on Advances in Signal Processing | Continuous 2D interaction-force regression across elbow postures | [Paper](https://doi.org/10.1186/s13634-024-01183-7) |

### Posture-Informed Muscular Force Learning for Robust Hand Pressure Estimation

Seo et al. propose PiMForce, which conditions forearm-sEMG features on 3D hand-joint information so the model can distinguish similar muscle activations produced in different hand postures. Their data system synchronizes an sEMG armband, markerless finger tracking, and a pressure glove for 21 participants across varied hand-object interactions, producing spatial pressure estimates over the fingertips and palm. The work shows that posture is not merely auxiliary metadata but a powerful conditioning signal for resolving the many-to-one relationship between EMG and exerted pressure.

### Force Estimation for Human-Robot Interaction Using Electromyogram Signals from Varied Arm Postures

Sittiruk et al. compare 19 variants of Gaussian-process, neural-network, linear, and support-vector regression using 250 ms RMS features from an eight-channel Myo armband to estimate planar interaction forces under three elbow-position scenarios. Exponential-kernel Gaussian-process regression performs best, with reported RMSE increasing from 1.18 N in the most constrained posture to 1.77 N when the elbow is free, clearly illustrating the cost of posture variability. Because the experiment includes only two participants, its strongest contribution is the controlled model-and-posture comparison rather than evidence for cross-user generalization.

## Contributing

When adding a paper, prefer the version-of-record or an open-access primary source, record the publication year and venue, link public code/data when available, and add a concise paragraph covering the method, evaluation setting, main result, and important limitation. Keep entries restricted to 2024 or later unless an older work is essential background.
