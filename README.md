<div align="center">

# ROVER

### Recovering the Realised Objective of a World-Model Controller for Underwater Vehicles

An auditable framework for translating the latent reward learned by a DreamerV3 controller into explicit, physically meaningful objective terms.

[![World Model](https://img.shields.io/badge/World_Model-DreamerV3-5b5bd6.svg)](#method)
[![Surrogate](https://img.shields.io/badge/Surrogate-Additive_KAN-0f766e.svg)](#method)
[![Application](https://img.shields.io/badge/Application-Underwater_Robotics-0369a1.svg)](#experimental-setting)

</div>

## Overview

World-model reinforcement learning trains a policy on trajectories imagined by a learned dynamics model. In this setting, the controller is optimized against a learned reward head operating on latent states—not directly against the reward function written by an engineer.

**ROVER** (Realised-Objective Verification for controllers synthesised inside a learned model) recovers that learned objective and rewrites it as an auditable specification over named physical quantities such as tracking error, angular velocity, vehicle attitude, and thruster commands.

The framework answers two questions in sequence:

1. **Is the learned reward grounded?** Compare the reward imagined by the deployed world model with the reward returned by the environment.
2. **What objective was actually learned?** Fit a strictly additive Kolmogorov-Arnold surrogate over physically interpretable variables and compare its terms with the written engineering reward.

<p align="center">
  <img src="assets/reward-reconstruction.png" width="900" alt="Environment reward, DreamerV3 imagined reward, and additive KAN reconstruction over a held-out episode">
</p>

## Key Results

| Question | Held-out result |
|---|---:|
| Learned reward vs. environment reward | $R^2 = 0.957$ |
| Additive recovery vs. learned reward | $R^2 = 0.881$ |
| Maximum decomposition residual | $1.2 \times 10^{-7}$ |
| Objective associated with tracking error | 41.5% |
| Objective associated with actuation | 19.6% |
| Objective matched term-by-term to the written reward | 83.6% |
| Objective assigned to unspecified velocity and episode-time terms | 16.4% |

The recovered objective is dominated by tracking quality and actuation. Importantly, it also exposes dependencies on **vehicle velocity** and **elapsed episode time**, neither of which appears as a term in the written specification.

<p align="center">
  <img src="assets/objective-share.png" width="900" alt="Recovered objective share corresponding to written and unspecified reward terms">
</p>

## Method

### 1. Recover the reward used in imagination

Recorded episodes are replayed through the deployed DreamerV3 checkpoint. For each recorded transition, ROVER advances the model prior under the recorded action and estimates the expected reward assigned by the learned reward head:

$$
y_t = \mathbb{E}_{\hat{s}_{t+1} \sim p_\phi}
\left[r_\phi(\hat{s}_{t+1}) \mid s_t, a_t\right].
$$

The resulting target is evaluated against the environment reward on held-out episodes before any interpretation is attempted.

### 2. Express the objective in named physical quantities

ROVER maps recorded observations and actions to 45 quantities in four groups:

| Group | Examples |
|---|---|
| Instantaneous state | Tracking error, velocity error, reference acceleration, angular rate, gravity direction, episode phase |
| Actuation | Six thruster commands and six command increments |
| Causal state history | Recent and persistent tracking error, velocity innovation, recent angular rate |
| Causal action history | Recent mean thruster commands |

All history features are causal and reset at episode boundaries. No recovered variable is read from the controller's latent state.

### 3. Fit a strictly additive KAN surrogate

The realized objective is represented as

$$
\hat{r}(x) = b + \sum_{i=1}^{45} \psi_i(x_i),
$$

where every $\psi_i$ is a learned univariate spline law over one named quantity. Because there is no hidden layer, each term is directly plottable, signed, reference-free, and the terms sum algebraically to the reported objective.

### 4. Audit the written specification

The recovered laws are compared with the additive projection of the non-additive engineering reward. This accounts for norm-coupled tracking terms and gated attitude or angular-rate bonuses, allowing deviations to be interpreted as concrete findings rather than surrogate artifacts.

## Experimental Setting

| Item | Configuration |
|---|---|
| Platform | Six-thruster BlueROV-class underwater vehicle |
| Task | Continuous lemniscate trajectory tracking |
| Controller | Deployed DreamerV3 checkpoint |
| Latent state | 256 deterministic + 1,024 categorical coordinates |
| Recorded evidence | 128 episodes / 49,575 transitions |
| Control interval | 16 ms |
| Train / validation / test episodes | 90 / 19 / 19 |
| Prior samples per transition | 16 |
| Recovered variables | 45 |
| KAN spline grid | Cubic splines, 8 grid intervals |

The deployed controller and its recorded trajectories remain unchanged throughout recovery. Training and evaluation are separated by complete episodes rather than individual transitions.

## What ROVER Produces

- A grounded estimate of the reward optimized inside the world model.
- A complete additive objective over logged physical variables.
- Per-variable contribution magnitudes and signed response curves.
- Threshold clauses stated in engineering units with held-out activation rates.
- A term-by-term comparison between the realized and written objectives.
- An auditable evidence trail tied to the deployed checkpoint and recorded episodes.

## Main Finding

> A learned world-model controller can be delivered with a readable statement of what it was actually trained to optimize—not only a report of the behavior it happened to produce.

For the evaluated controller, 83.6% of the recovered objective corresponds to the written reward. The remaining 16.4% reveals learned dependence on quantities omitted from the specification, providing a concrete target for engineering review and future online monitoring.

## Citation

If you use this work, please cite the preprint:

```bibtex
@article{zhang2026rover,
  title   = {ROVER: Recovering the Realised Objective of a World-Model Controller for Underwater Vehicles},
  author  = {Zhang, Tianze and Wu, Lei and Lou, Min and Yu, Xiang and Tao, Xiaowen},
  year    = {2026},
  note    = {Submitted to Ocean Engineering}
}
```
---

<div align="center">
  <sub>World-model control · Objective recovery · Additive KAN · Auditable autonomy · Underwater robotics</sub>
</div>
