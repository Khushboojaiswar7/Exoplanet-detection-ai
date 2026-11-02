AI-Enabled Exoplanet Detection from Noisy Astronomical Light Curves
Project status: Proposed solution / hackathon concept. The
architecture, methods, and interface described here are a design based
on the project presentation. Update this README to distinguish
implemented components from planned components before publishing it as
a completed software project.

Overview
This project proposes an AI-assisted pipeline for identifying and
characterizing potential exoplanet transits in noisy astronomical light
curves. It is designed to reduce the number of candidates that need
expensive downstream analysis, classify promising signals into four
categories, and estimate transit parameters with uncertainty
information.

The proposed workflow uses a fast first-stage screening method, a
multimodal classification model that combines light-curve views with
engineered features, and Bayesian-style parameter fitting for confirmed
candidates.
Problem
Astronomical light curves can contain noise, stellar variability,
instrumental artifacts, and signals that resemble planetary transits. A
simple transit/non-transit classifier may miss important distinctions
between genuine exoplanet-like signals and other astrophysical events.
Computationally expensive modelling of every light curve is also
inefficient.
Proposed Solution
The system is organized into three stages:
1. Accelerated candidate screening: Apply Box Least Squares (BLS)
   period search to prioritize light curves with promising periodic
   dips.
2. Four-class multimodal classification: Phase-fold the candidate
   light curves, prepare global and local views, extract engineered
   BLS/centroid features, and combine the inputs in a CNN-plus-tabular
   model.
3. Transit parameter estimation: For selected candidates, use MCMC
   posterior sampling with a Mandel--Agol transit model to estimate
   physical transit parameters and their uncertainty.
The final output is intended to include a predicted class, confidence
estimate, detection statistics, parameter estimates, plots, and a
concise technical report.
Intended Classification Labels
- Exoplanet Transit --- a signal consistent with a planetary
  transit.
- Eclipsing Binary --- a signal more consistent with an eclipsing
  binary system.
- Blended Signal --- a possible transit-like signal affected by
  blended sources.
- Other Astrophysical --- a signal better explained by another
  astrophysical phenomenon.
These are proposed output categories; their definitions and training
labels should be aligned with the chosen dataset and scientific
validation protocol.
Pipeline
TESS light-curve data
        |
        v
Preprocessing
(missing values, outliers, normalization, detrending)
        |
        v
GPU-accelerated BLS screening
        |
        v
Candidate selection
   |             |
   | no          | yes
   v             v
Reject /      Phase folding and feature extraction
deprioritize  (global view, local view, BLS and centroid features)
                      |
                      v
             Multimodal fusion model
             (CNN + tabular features)
                      |
                      v
             Four-class prediction
                      |
                      v
             Candidate parameter fitting
             (MCMC + transit model)
                      |
                      v
             Confidence, uncertainty,
             visualizations and report
Key Features
1. GPU-Accelerated BLS Screening
Uses Box Least Squares period search to narrow a large set of light
curves to a smaller candidate pool before more expensive modelling.
2. Multimodal Feature Fusion
Combines phase-folded global and local light-curve views with engineered
features, such as BLS statistics and centroid-shift measurements.
3. Four-Class Classification
Goes beyond a binary transit/non-transit decision by predicting one of
four intended signal categories.
4. MC-Dropout Confidence Estimation
Uses repeated stochastic forward passes at inference time to estimate
predictive variability. This should be evaluated for calibration and
should not be described as a guaranteed probability of correctness.
5. MCMC Transit-Parameter Fitting
Uses posterior sampling to estimate quantities such as orbital period,
transit depth, and transit duration, with credible intervals where
supported by the model and data.
6. Baseline Comparison
Proposes training a Random Forest baseline on tabular features in
parallel, so the fusion model can be compared against a simpler approach
using consistent evaluation data and metrics.
7. Scientific Visualization and Reporting
Plans to generate diagnostic plots, annotated light curves, parameter
summaries, uncertainty intervals, and a technical report for each
candidate.
Proposed Technology Stack
  Component               Proposed tools          Purpose
  Data access and         astropy, lightkurve Access and process
light-curve handling                            astronomical light
curves

  Numerical processing    NumPy, SciPy,       Data manipulation and
pandas                scientific computation

  Candidate screening     BLS implementation, GPU Search for periodic
acceleration where      transit-like dips
available               

  Deep learning           PyTorch or            CNN and multimodal
TensorFlow            model

  Baseline model          scikit-learn          Random Forest and
evaluation

  Transit modelling       batman /              Transit light-curve
batman-package        model

  Posterior sampling      emcee                 MCMC parameter
estimation
Visualization           Matplotlib            Diagnostic plots and
result visualization
Implementation note: The presentation lists some packages and
methods as alternatives. Select and verify the final library/API
versions during implementation. GPU support and package compatibility
depend on the runtime environment.
Data Sources
Potential data sources shown in the proposal include:
- NASA TESS light-curve data.
- Public astronomical archives such as MAST.
- An appropriate labelled dataset for training and evaluating the
  classifier.
The exact dataset, catalogue version, sample selection, licensing, and
train/validation/test split should be documented before experiments are
reported. Any "ISRO-provided labelled set" should be mentioned only if
access and permission have actually been confirmed.
Evaluation Plan
The proposed system should be evaluated using a fixed, documented
dataset split and appropriate baselines.
- Classification: macro-F1, per-class precision/recall, confusion
  matrix, and balanced accuracy.
- Probability quality: reliability diagrams, Brier score or
  another suitable calibration metric.
- Candidate screening: recall of known candidates, false-positive
  rate, and compute time saved.
- Parameter estimation: compare estimates and uncertainty
  intervals against validated catalogue values or injected synthetic
  signals.
- Baseline comparison: evaluate the CNN-plus-tabular model and
  Random Forest using the same held-out examples.
- Robustness: test across sectors, noise levels, and different
  types of stellar variability.
Avoid claiming scientific accuracy, calibrated confidence, or
computational speedups until these are measured experimentally.
Estimated Cost
The presentation estimates that a hackathon-scale version could
potentially use free public data, open-source libraries, and free-tier
notebook compute. It lists a possible additional compute budget of up
to approximately ₹1,500 if free GPU limits are exceeded.

This is a planning estimate, not a guaranteed cost. Actual spending
depends on dataset size, runtime availability, GPU access, and training
duration. MCMC can be CPU-bound, but its cost depends on the number of
candidates and sampling configuration.
Current Scope and Next Steps
This document describes the proposed design. Before presenting the
project as implemented, complete and document the following:
- [ ] Confirm dataset source and labels.
- [ ] Implement and test preprocessing.
- [ ] Implement BLS candidate screening.
- [ ] Build reproducible global/local phase-folded inputs.
- [ ] Train and validate the four-class model.
- [ ] Compare against a Random Forest baseline.
- [ ] Evaluate confidence calibration.
- [ ] Implement and validate transit-parameter fitting.
- [ ] Add plots, a reproducible report, and example outputs.
- [ ] Record metrics, compute environment, limitations, and
  references.
Limitations and Scientific Caution
- A transit-like dip alone does not establish that an exoplanet
  exists.
- Model confidence is not the same as scientific confirmation.
- Data leakage can inflate performance; splits should account for the
  dataset structure and, where appropriate, separate targets or
  sectors.
- Centroid shifts, stellar variability, and instrumental artifacts
  require careful treatment.
- Candidate classifications and fitted parameters should be validated
  against known sources or carefully designed injection-and-recovery
  experiments.
- This tool should be treated as a candidate-triage and
  research-support system, not an autonomous confirmation system.
Suggested Repository Structure
exoplanet-detection/
├── README.md
├── data/                 # Data instructions only; avoid committing large/raw data
├── notebooks/            # Exploratory analysis and experiments
├── src/
│   ├── preprocessing/
│   ├── screening/
│   ├── features/
│   ├── models/
│   ├── parameter_fitting/
│   └── visualization/
├── tests/
├── reports/
├── requirements.txt
└── LICENSE
Acknowledgements and References
When implementation begins, cite the specific TESS/MAST data products,
relevant BLS method reference, transit-model documentation, MCMC library
documentation, and any labelled datasets used. Add contributors and
acknowledgements only with their consent.
