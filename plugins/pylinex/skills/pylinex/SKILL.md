# Pylinex — Global 21cm Signal Extraction Toolkit

You are helping a user write Python code using the **pylinex** library ecosystem for global 21cm cosmological signal extraction. This skill gives you complete knowledge of the library API so you can generate correct, idiomatic code.

## Library Overview

The ecosystem consists of four libraries that work together:

- **pylinex** — The user-facing library. Performs linear and nonlinear signal extractions with classes for models, likelihoods, fitters, and basis representations.
- **distpy** — Probability distribution library. Provides distributions, transforms, jumping distributions (MCMC proposals), and the Metropolis-Hastings sampler that pylinex uses.
- **ares** — Accelerated Reionization Era Simulations. Physics-based simulations of the 21cm signal, reionization, and galaxy populations.
- **perses** — Parameterized Radio Sky Simulator. Foreground models, beam simulations, and the full 21cm observation model.

Users primarily write code against **pylinex**. The other three are dependencies that surface when users need to configure priors (distpy), generate training sets from physical models (ares), or model instrument effects (perses).

## Installation

```bash
# Recommended: use a conda environment
conda create -n pylinex_env -c conda-forge -y python=3.13 pip numpy scipy matplotlib healpy h5py sympy pandas scikit-learn numdifftools
conda activate pylinex_env

# Clone and install each library
git clone https://github.com/caden-kunkel/distpy.git
cd distpy && pip install -e . && cd ..

git clone https://github.com/caden-kunkel/pylinex.git
cd pylinex && pip install -e . && cd ..

git clone https://github.com/caden-kunkel/perses.git
cd perses && pip install -e . && cd ..

git clone https://github.com/caden-kunkel/ares.git
cd ares && pip install -e . && cd ..
```

If `pip install -e .` doesn't place packages correctly, manually copy:
```bash
DST=$(python -c "import site; print(site.getsitepackages()[0])")
cp -R distpy/distpy $DST/
cp -R pylinex/pylinex $DST/
cp -R perses/perses $DST/
cp -R ares/ares $DST/
```

Dependencies: numpy, scipy, matplotlib, h5py, healpy, sympy, pandas, scikit-learn, numdifftools. Optional: emcee (for ensemble sampling).

---

## Core Workflow

The typical pylinex analysis follows this pipeline:

1. **Prepare training sets** — arrays of simulated curves (from ares/perses or custom) that span the expected signal/foreground space
2. **Create basis representations** — use `TrainedBasis` (SVD) to compress training sets into a few basis vectors
3. **Build models** — combine bases into `BasisModel`, compose with `SumModel`, apply `Expander` transforms
4. **Fit data** — use `Fitter` (analytic linear), `Extractor` (grid search over truncations), or `Sampler` (MCMC)
5. **Analyze results** — posterior distributions, residuals, component separation

---

## Module Reference

### pylinex.basis

Basis vectors for representing signals and foregrounds in compressed form.

#### Basis (base class)
```python
Basis(basis_vectors, expander=None, translation=None)
```
- `basis_vectors`: (k, N) ndarray — k basis vectors of length N
- `expander`: optional Expander to apply after evaluation
- `translation`: (N,) constant offset added to output

Key properties/methods:
- `basis` — raw (k, N) matrix
- `expanded_basis` — basis after expander applied
- `num_basis_vectors` — k
- `__call__(parameters)` — evaluate: returns translation + parameters @ basis
- `__getitem__(slice)` — take subset of basis vectors
- `__add__(other)` — concatenate bases
- `projection_onto_basis(data, error=None)` — project data onto basis space
- `RMS_of_training_set_fits(training_set, error=None)` — evaluate quality

#### TrainedBasis
```python
TrainedBasis(training_set, num_basis_vectors, error=None,
             expander=None, mean_translation=False)
```
- `training_set`: (ncurves, nchannels) ndarray of simulated curves
- `num_basis_vectors`: how many SVD modes to keep
- `error`: (nchannels,) weighting for weighted SVD
- `mean_translation`: if True, subtract and store training set mean

Derives basis vectors via SVD of the (optionally weighted) training set. The resulting basis captures the dominant modes of variation.

Key properties:
- `full_importances` — singular values (importance of each mode)
- `training_set_space_singular_vectors` — full SVD result

#### BasisSum
```python
BasisSum(names, bases)
```
- `names`: list of string identifiers (e.g., ['signal', 'foreground'])
- `bases`: list of Basis objects

Container that concatenates multiple named bases. Used throughout pylinex to represent multi-component models. The combined parameter vector is the concatenation of all component parameters.

Key methods:
- `basis_dot_products(error=None)` — cross-correlation matrix between components
- `basis_subsets(**subsets)` — create truncated BasisSum (e.g., `basis_sum.basis_subsets(signal=3, foreground=5)`)

#### Other Basis Classes
- `PolynomialBasis(xs, num_basis_vectors)` — polynomial basis over x values
- `FourierBasis(xs, num_basis_vectors)` — Fourier basis
- `LegendreBasis(xs, num_basis_vectors)` — Legendre polynomial basis
- `GramSchmidtBasis(basis_vectors, error=None)` — orthonormalized basis

#### Utility Functions
- `effective_training_set_rank(training_set, error=None)` — estimate intrinsic dimensionality
- `plot_training_set_with_modes(training_set, basis, error=None)` — diagnostic visualization

---

### pylinex.model

Models map parameter vectors to output spectra. All models implement a common interface.

#### Model (abstract base)
All models provide:
- `num_channels` — output vector length
- `num_parameters` — input parameter count
- `parameters` — list of parameter name strings
- `__call__(parameters)` — evaluate model: (num_parameters,) → (num_channels,)
- `gradient(parameters)` — Jacobian: (num_channels, num_parameters)
- `hessian(parameters)` — (num_channels, num_parameters, num_parameters)
- `gradient_computable` / `hessian_computable` — bool flags

#### BasisModel
```python
BasisModel(basis)
```
The most common model type. Wraps a Basis (or BasisSum) as a Model. Parameters are the basis coefficients. Output is the linear combination of basis vectors.

- Parameters named: `'a0'`, `'a1'`, ..., `'a{k-1}'`
- Gradient is simply the basis matrix (constant)

#### Parametric Signal Models
```python
GaussianModel(x_values)      # Gaussian absorption/emission profile
SechModel(x_values)           # Hyperbolic secant profile
LorentzianModel(x_values)     # Lorentzian profile
TanhModel(x_values)           # Tanh-based turning point model
SinusoidalModel(x_values)     # Sinusoidal model
```
Each takes frequency/x values and has physically motivated parameters (amplitude, center, width, etc.).

#### Composite Models

**SumModel** — sum of sub-model outputs (most common composite):
```python
SumModel(names, models)
# e.g., SumModel(['signal', 'foreground'], [signal_model, fg_model])
```
Output = model1(pars1) + model2(pars2). Parameters are concatenated.

**ProductModel** — element-wise product of sub-model outputs:
```python
ProductModel(names, models)
```

**DirectSumModel** — vertically stacks outputs (for multi-band data):
```python
DirectSumModel(names, models)
# num_channels = sum of sub-model num_channels
```

**CompositeModel** — function composition (output of one feeds into another):
```python
CompositeModel(names, models)
```

#### Modification Models

```python
ExpandedModel(model, expander)        # Apply Expander to output
ScaledModel(model, scale_factor)      # Multiply output by scalar
TransformedModel(model, transform)    # Apply distpy Transform to output
ShiftedRescaledModel(model, shift, rescale)
DistortedModel(model, distortion_matrix)
BinnedModel(model, binner)            # Bin output channels
ProjectedModel(model, projection_matrix)
RenamedModel(model, new_parameter_names)
RestrictedModel(model, fixed_parameters)  # Fix some parameters
SlicedModel(model, output_slice)      # Take subset of output channels
```

#### Advanced Models
- `BasisFitModel(basis, data, error)` — pre-fitted basis model (fixed parameters)
- `ConditionalFitModel` — parameters fitted conditional on other parameters
- `ExpressionModel(expression, num_channels, parameters)` — model from string expression
- `FixedModel(curve)` — model with no parameters, returns fixed curve
- `ConstantModel(num_channels)` — single parameter 'a', constant output
- `TruncatedBasisHyperModel` — hypermodel over truncation levels
- `EmulatedModel` — neural network emulator of expensive model
- `InputInterpolatedModel` / `OutputInterpolatedModel` — interpolation-based models

#### Model Utilities
- `TrainingSetCreator` — generates training sets from Model objects
- `ModelTree` — hierarchical model visualization
- `load_model_from_hdf5_group(group)` — load saved model

---

### pylinex.expander

Expanders transform vectors between spaces (e.g., from a small basis space to the full data space). They are memory-efficient alternatives to storing full matrices.

#### Expander (abstract base)
```python
apply(vector)                          # expand vector
expansion_matrix(original_space_size)  # get explicit matrix form
expanded_space_size(original_space_size)
original_space_size(expanded_space_size)
overlap(vectors, error=None)           # compute Ψᵀ C⁻¹ y
```

#### Concrete Expanders
```python
NullExpander()                          # identity (no-op)
PadExpander(original_size, expanded_size, pad_value=0)  # zero-pad
IndexExpander(indices, expanded_size)   # select/reorder indices
RepeatExpander(factor)                  # repeat each element
MultipleExpander(factor)                # multiply by scalar
ModulationExpander(modulation_curve)    # element-wise multiply
MatrixExpander(matrix)                  # arbitrary matrix transform
CompositeExpander(expanders)            # chain multiple expanders
ShapedExpander(expander, shape)         # reshape output
DerivativeExpander(x_values)            # numerical differentiation
AxisExpander(axis, expanded_num_axes)   # expand along axis
```

#### ExpanderSum
```python
ExpanderSum(expanders)
```
Applies different expanders to different components and sums results. Used internally by BasisSum/Fitter when components have different expanders.

---

### pylinex.fitter

Linear fitting and model selection via analytic solutions.

#### Fitter
```python
Fitter(basis_sum, data, error=None, **priors)
```
- `basis_sum`: BasisSum (or single Basis, auto-wrapped)
- `data`: (nchannels,) or (ncurves, nchannels) ndarray
- `error`: (nchannels,) noise levels or covariance matrix
- `**priors`: keyword args like `signal_prior=GaussianDistribution(mean, cov)` — one per named basis component

Performs weighted least-squares with optional Gaussian priors. Solution is analytic (no iteration).

Key properties:
- `fit_parameters` — best-fit coefficient vector
- `fit_error_covariance` — posterior parameter covariance
- `fit_residuals` — data minus best-fit model
- `channel_RMS` — RMS of residuals
- `channel_bias` — mean of residuals
- `log_posterior_likelihood` — log evidence
- `posterior_distribution` — distpy GaussianDistribution
- `data_significance` — chi-squared of data term
- `prior_significance` — chi-squared of prior term

When `data` is 2D (multiple curves), all properties become arrays over curves.

#### MetaFitter
```python
MetaFitter(basis_sum, data, error, compiled_quantity, quantity_to_minimize,
           *dimensions, **priors)
```
- `compiled_quantity`: CompiledQuantity with metrics to evaluate at each grid point
- `quantity_to_minimize`: string name of metric to minimize
- `*dimensions`: grid specification — list of dicts mapping component names to truncation arrays

Runs a Fitter at every point on a truncation grid. Used for model selection (finding optimal number of basis vectors per component).

Key properties:
- `grids` — computed quantity values over full grid
- `optimal_indices` — grid indices that minimize chosen quantity
- `shape` — grid shape

#### Extractor
```python
Extractor(data, error, names, training_sets, dimensions,
          compiled_quantity=CompiledQuantity('empty'),
          quantity_to_minimize='bias_score', expanders=None,
          mean_translation=False, num_curves_to_score=None,
          use_priors_in_fit=False, prior_covariance_expansion_factor=1.,
          prior_covariance_diagonal=False, verbose=True)
```
- `names`: list of component names (e.g., ['signal', 'foreground'])
- `training_sets`: list of (ncurves, nchannels) arrays, one per component
- `dimensions`: grid of truncation levels to search

End-to-end linear extraction: creates TrainedBasis for each component, builds BasisSum, runs MetaFitter grid search, selects optimal truncations.

Key methods:
- `run()` — execute full extraction pipeline
- `save(file_name)` / `load(file_name)` — HDF5 persistence

#### Other
- `TrainingSetIterator` — memory-efficient iteration over large training sets
- `MAAFitter` — Mean Absolute Amplitude variant

---

### pylinex.loglikelihood

Likelihood functions for Bayesian inference.

#### GaussianLoglikelihood
```python
GaussianLoglikelihood(data, error, model)
```
- `data`: (nchannels,) observed data
- `error`: (nchannels,) noise standard deviations or covariance matrix
- `model`: Model object

Computes: ln L = -0.5 * (data - model(θ))ᵀ C⁻¹ (data - model(θ))

Key methods:
- `__call__(parameters)` — evaluate log-likelihood
- `gradient(parameters)` �� gradient w.r.t. parameters
- `hessian(parameters)` — Hessian matrix

#### Other Likelihoods
- `PoissonLoglikelihood(data, model)` — for count data
- `GammaLoglikelihood(data, model, shape)` — Gamma-distributed noise
- `LinearTruncationLoglikelihood` — penalizes basis truncation
- `NonlinearTruncationLoglikelihood` — nonlinear truncation penalty
- `ConditionalFitGaussianLoglikelihood` — Gaussian with conditional fits
- `LikelihoodDistributionHarmonizer` — combines multiple likelihoods
- `RosenbrockLoglikelihood` — test function for sampler validation

---

### pylinex.nonlinear

MCMC sampling and nonlinear fitting.

#### Sampler
```python
Sampler(file_name, num_walkers, loglikelihood,
        jumping_distribution_set=None, guess_distribution_set=None,
        prior_distribution_set=None, steps_per_checkpoint=100,
        verbose=True, restart_mode=None, num_threads=1,
        use_ensemble_sampler=False, desired_acceptance_fraction=0.25)
```
- `file_name`: HDF5 output path (chains saved here)
- `num_walkers`: number of MCMC walkers/chains
- `loglikelihood`: Loglikelihood object
- `jumping_distribution_set`: distpy JumpingDistributionSet (proposal)
- `guess_distribution_set`: distpy DistributionSet (initialization)
- `prior_distribution_set`: distpy DistributionSet (optional additional priors)
- `restart_mode`: None, 'continue', 'reinitialize', 'update', 'trimmed_update'
- `use_ensemble_sampler`: if True, uses emcee's EnsembleSampler

Key methods:
- `run(num_steps)` — run MCMC for given number of steps
- `close()` — close HDF5 file handle

#### NLFitter
```python
NLFitter(file_name, burn_rule=None, chunk_slice=slice(None))
```
- `file_name`: HDF5 file written by Sampler
- `burn_rule`: BurnRule specifying burn-in/thinning
- `chunk_slice`: which saved chunks to load

Loads and analyzes MCMC chains. Supports context manager (`with NLFitter(...) as f:`).

Key properties:
- `chain` — (nsamples, nparams) array of post-burn-in samples
- `loglikelihood_values` — ln L at each sample
- `acceptance_fraction` — overall acceptance rate
- `mean` — posterior mean vector
- `covariance` — posterior covariance matrix
- `posterior_distribution` — fitted GaussianDistribution

Key methods:
- `univariate_histogram(index, ...)` — 1D marginal plot
- `bivariate_histogram(index1, index2, ...)` — 2D marginal
- `triangle_plot(...)` — full corner plot

#### BurnRule
```python
BurnRule(min_checkpoints=None, desired_fraction=None, thin=None, burn_end=False)
```
- `desired_fraction`: fraction of chain to keep (0 to 1)
- `thin`: thinning stride
- `burn_end`: if True, burn from end instead of beginning

#### TruncationExtractor
```python
TruncationExtractor(data, error, names, training_sets, nterms_maxima,
                    file_name, information_criterion='deviance_information_criterion',
                    expanders=None, mean_translation=False, trust_ranks=False)
```
Nonlinear (MCMC-based) counterpart to Extractor. Samples over truncation levels as discrete parameters.

#### LeastSquareFitter
```python
LeastSquareFitter(loglikelihood, prior_distribution_set=None,
                  guess_distribution_set=None, transform_list=None, **bounds)
```
Deterministic optimizer (scipy.optimize) for finding maximum likelihood/posterior.

---

### pylinex.quantity

Lazy evaluation system for computing derived quantities from fitter states.

#### CompiledQuantity
```python
CompiledQuantity(name, *quantities)
```
Container for multiple Quantity objects. Passed to MetaFitter/Extractor to specify what metrics to compute at each grid point.

#### Quantity Subclasses
```python
ConstantQuantity(name, value)                    # fixed value
AttributeQuantity(name, attribute_name)          # get fitter.attribute
FunctionQuantity(name, function)                 # compute function(fitter)
CalculatedQuantity(name, function, *inputs)      # derive from other quantities
```

Common usage pattern:
```python
from pylinex import CompiledQuantity, AttributeQuantity, FunctionQuantity

compiled_qty = CompiledQuantity('metrics',
    AttributeQuantity('channel_RMS', 'channel_RMS'),
    AttributeQuantity('log_evidence', 'log_posterior_likelihood'),
    FunctionQuantity('bias_score', lambda fitter: compute_bias(fitter))
)
```

---

### pylinex.interpolator

```python
LinearInterpolator(x_values, y_values)
QuadraticInterpolator(x_values, y_values)
DelaunayLinearInterpolator(points, values)
```
Used internally for interpolated models and least-square fitters.

---

### pylinex.forecast

#### Forecaster
```python
Forecaster(file_name, num_curves_to_create, error, names, training_sets,
           input_curve_sets, dimensions, compiled_quantity, quantity_to_minimize,
           expanders=None, mean_translation=False, ...)
```
Runs Extractor on many simulated data realizations to forecast reconstruction quality. Saves results to HDF5.

---

### pylinex.hdf5

File-level loaders and visualization:
```python
load_quantity_from_hdf5_file(file_name, group_name)
load_expander_from_hdf5_file(file_name, group_name)
load_model_from_hdf5_file(file_name, group_name)
load_loglikelihood_from_hdf5_file(file_name, group_name)
```

`ExtractionPlotter` — visualization class for saved extraction results.

---

## distpy Reference (User-Facing Surfaces)

Users interact with distpy when setting up priors, proposals, and initial guesses for MCMC.

### Distributions (priors and initialization)

```python
from distpy import GaussianDistribution, UniformDistribution, DistributionSet

# Single distribution
prior = GaussianDistribution(mean, variance)  # 1D
prior = GaussianDistribution(mean_vector, covariance_matrix)  # multivariate

# Distribution set (maps parameter names to distributions)
dset = DistributionSet()
dset.add_distribution(GaussianDistribution(0, 1), 'amplitude')
dset.add_distribution(UniformDistribution(50, 200), 'center_freq')
```

Common distributions: `GaussianDistribution`, `UniformDistribution`, `TruncatedGaussianDistribution`, `ExponentialDistribution`, `GammaDistribution`, `ChiSquaredDistribution`, `BetaDistribution`, `InfiniteUniformDistribution` (improper uniform).

### Transforms

```python
from distpy import TransformList, LogTransform, AffineTransform, NullTransform

# Applied to parameters before/during sampling
transform_list = TransformList(LogTransform(), NullTransform(), AffineTransform(shift, scale))
```

Common transforms: `NullTransform`, `LogTransform`, `Log10Transform`, `AffineTransform`, `ExponentialTransform`, `ArsinhTransform`, `BoxCoxTransform`, `PowerTransform`, `ReciprocalTransform`, `CompositeTransform`.

### Jumping Distributions (MCMC proposals)

```python
from distpy import GaussianJumpingDistribution, JumpingDistributionSet

# Proposal distribution for MCMC
jds = JumpingDistributionSet()
jds.add_distribution(GaussianJumpingDistribution(covariance_matrix), parameter_names)
```

### MetropolisHastingsSampler
Low-level sampler (usually accessed through pylinex's Sampler wrapper):
```python
from distpy import MetropolisHastingsSampler
```

---

## ares Reference (Training Set Generation)

Users interact with ares to generate physically-motivated 21cm signal training sets.

### Key Classes
```python
import ares

# Run a global 21cm simulation
sim = ares.simulations.Global21cm(**parameters)
sim.run()
frequencies = sim.history['nu']  # MHz
signal = sim.history['dTb']      # brightness temperature (mK)
```

### Common Simulation Parameters
- `fX`: X-ray efficiency
- `fstar`: star formation efficiency
- `Tmin`: minimum halo virial temperature
- `Nlw`: Lyman-Werner photon count
- Custom parameter bundles available via `ares.util.ParameterBundles`

### Generating Training Sets
```python
# Vary parameters to create training set
import numpy as np

training_curves = []
for params in parameter_samples:
    sim = ares.simulations.Global21cm(**params)
    sim.run()
    training_curves.append(sim.history['dTb'])

training_set = np.array(training_curves)  # (ncurves, nchannels)
```

---

## perses Reference (Foreground, Signal, and Instrument Models)

Users interact with perses for foreground basis models, signal models, beam effects, and full observation simulation.

### Foreground Basis Models (most commonly used)

These are the primary foreground models used with pylinex. They produce `BasisModel` objects directly.

```python
from perses.models import PowerLawTimesLogPolynomialModel, PowerLawTimesPolynomialModel, LogLogPolynomialModel

# EDGES-style foreground model (power law × polynomial in log-frequency)
fg_model = PowerLawTimesLogPolynomialModel(x_values=frequencies, num_terms=10,
                                            spectral_index=-2.5)
fg_basis = fg_model.basis  # extract the Basis object for use with BasisSum/Fitter

# Power law × polynomial (linear frequency)
fg_model2 = PowerLawTimesPolynomialModel(x_values=frequencies, num_terms=5)

# Log-log polynomial
fg_model3 = LogLogPolynomialModel(x_values=frequencies, num_terms=5)
```

Parameters:
- `x_values`: frequency array (MHz)
- `num_terms`: number of polynomial terms (basis vectors)
- `spectral_index`: power law index (default -2.5)
- `reference_x`: normalization frequency (default: mean of min/max)

### Signal Models

```python
from perses.models import Tanh21cmModel, FlattenedGaussianModel, TurningPointModel, FourParameterModel

# Tanh-based 21cm model (absorption trough)
signal_model = Tanh21cmModel(x_values=frequencies)

# Flattened Gaussian (EDGES-style parameterization)
signal_model = FlattenedGaussianModel(x_values=frequencies)

# Turning point model
signal_model = TurningPointModel(x_values=frequencies)
```

### Spectral Index Models (for training set generation)

```python
from perses.models import GaussianSpectralIndexModel, ConstantSpectralIndexModel

model_index = GaussianSpectralIndexModel()
model_index.nside = 20  # HEALPix resolution
```

### Foreground Sky Models

```python
from perses.foregrounds import Galaxy, HaslamGalaxy, GSMGalaxy, SpatialPowerLawGalaxy
```
- `HaslamGalaxy` — Haslam 408 MHz map-based sky
- `GSMGalaxy` — Global Sky Model
- `SpatialPowerLawGalaxy` — parametric power-law foreground

### Beam Models

```python
from perses.beam.polarized.GaussianDipoleBeam import GaussianDipoleBeam

# Beam with frequency-dependent FWHM
beam = GaussianDipoleBeam(lambda nu: ct + lt*nu + qt*nu**2)
```

### Observation Simulation

```python
from perses.simulations import GroundObservatory, UniformDriftscanSetCreator

location = GroundObservatory(latitude=-79.82, longitude=38.43, angle=0)
```

### Full Observation Model
```python
from perses.models import Full21cmModel, ThroughReceiverModel
```
Combines signal, foreground, beam, and receiver into a complete observation model.

---

## Tutorial-Based Workflows

### Problem 1: Simple Polynomial Foreground Fit
```python
import h5py, numpy as np
from pylinex import Fitter, BasisSum, PolynomialBasis

# Load data
with h5py.File('data.hdf5', 'r') as f:
    frequencies = f['frequencies'][()]
    noise_level = f['brightness_temperature_noise_level'][()]
    temperatures = f['brightness_temperatures'][()]

# Polynomial fit
basis = PolynomialBasis(xs=frequencies, num_basis_vectors=5)
basis_sum = BasisSum(names=['polynomial'], bases=[basis])
fitter = Fitter(basis_sum=basis_sum, data=temperatures, error=noise_level)

print(f'RMS residual = {fitter.channel_bias_RMS:.2f} K')
fitter.plot_subbasis_fit(nsigma=1, name='polynomial', x_values=frequencies,
    subtract_truth=True, true_curve=temperatures, colors='r',
    xlabel=r'$\nu$ (MHz)', ylabel='Temperature Residual (K)', show=True)
```

### Problem 2: MetaFitter with Information Criterion Optimization
```python
from perses.models import PowerLawTimesLogPolynomialModel
from pylinex import MetaFitter, AttributeQuantity

# Use physically-motivated foreground basis
fg_model = PowerLawTimesLogPolynomialModel(x_values=frequencies, num_terms=10)
fg_basis = fg_model.basis
basis_sum = BasisSum(names=['foreground'], bases=[fg_basis])

# Optimize number of terms via BPIC
quantity = AttributeQuantity('BPIC')
dimension = [{'foreground': np.arange(1, 11)}]

meta_fitter = MetaFitter(basis_sum, temperatures, noise_level,
                         quantity, 'BPIC', dimension)

# Get optimal fitter
optimal_fitter = meta_fitter.fitter_from_indices(meta_fitter.optimal_indices)
```

### Problem 3: Two-Component Extraction (Foreground + Signal)
```python
from perses.models import PowerLawTimesLogPolynomialModel
from pylinex import Basis, BasisSum, MetaFitter, AttributeQuantity
import numpy as np

# Foreground model
fg_model = PowerLawTimesLogPolynomialModel(x_values=frequencies, num_terms=20)
fg_basis = fg_model.basis

# Signal model (e.g., Gaussian absorption feature as single basis vector)
# Define a Gaussian signal template
center, width, amplitude = 78.0, 10.0, -0.5
signal_template = amplitude * np.exp(-0.5 * ((frequencies - center)/width)**2)
signal_basis = Basis(signal_template[np.newaxis, :])  # shape (1, nchannels)

# Combined fit
basis_sum = BasisSum(names=['foreground', 'signal'], bases=[fg_basis, signal_basis])
quantity = AttributeQuantity('BPIC')
dimension = [{'foreground': np.arange(1, 11), 'signal': np.array([1])}]

meta_fitter = MetaFitter(basis_sum, temperatures, noise_level,
                         quantity, 'BPIC', dimension)
```

### Problem 4: Multi-Spectrum Data with Expanders
```python
from pylinex import (TrainedBasis, BasisSum, MetaFitter, AttributeQuantity,
                     RepeatExpander, NullExpander)

# When data has multiple spectra (e.g., multiple time bins),
# flatten everything to 1D and use expanders
nspectra = 10
nfreq = len(frequencies)

temperatures_flat = temperatures.flatten()   # (nspectra * nfreq,)
noise_level_flat = noise_level.flatten()

# Foreground: independent per spectrum → NullExpander (already spans full space)
# Signal: same signal in every spectrum → RepeatExpander
fg_basis = ...  # full-size basis spanning all spectra
signal_basis = TrainedBasis(signal_training_set, num_basis_vectors=5)

basis_sum = BasisSum(
    names=['foreground', 'signal'],
    bases=[Basis(fg_basis_vectors, expander=NullExpander()),
           Basis(signal_basis.basis, expander=RepeatExpander(nspectra))]
)
```

### Problem 5: TrainedBasis from Signal Training Set
```python
from perses.models import PowerLawTimesLogPolynomialModel
from pylinex import TrainedBasis, BasisSum, MetaFitter, AttributeQuantity

# Load training set of simulated 21cm signals (e.g., tanh models)
with h5py.File('signal_training_set.hdf5', 'r') as f:
    signal_training_set = f['signal_training_set'][()]  # (ncurves, nfreq)

# Create SVD-based signal basis
signal_basis = TrainedBasis(signal_training_set, num_basis_vectors=10,
                            error=noise_level, mean_translation=True)

# Foreground basis
fg_model = PowerLawTimesLogPolynomialModel(frequencies, 10)
fg_basis = fg_model.basis

# Combined extraction
basis_sum = BasisSum(['foreground', 'signal'], [fg_basis, signal_basis])
quantity = AttributeQuantity('BPIC')
dimension = [{'foreground': np.arange(1, 11), 'signal': np.arange(1, 11)}]
```

### Problem 6: Extractor with Beam-Weighted Foreground
```python
from pylinex import Extractor, CompiledQuantity, AttributeQuantity
from pylinex import NullExpander, RepeatExpander

# Multi-bin data with beam effects
ncurves, nbins, nfreq = 100, 10, len(frequencies)

# Flatten multi-dimensional data
temperatures_flat = temperatures.flatten()          # (nbins * nfreq,)
noise_level_flat = noise_level.flatten()
fg_training_flat = foreground_training_set.reshape(ncurves, -1)  # (ncurves, nbins*nfreq)

# Expanders: foreground covers full space, signal repeats across bins
expander_list = [NullExpander(), RepeatExpander(nbins)]

# Run Extractor
dimension = [{'foreground': np.arange(1, 11)}, {'signal': np.arange(1, 11)}]
quantity = CompiledQuantity('compiled', AttributeQuantity('BPIC'))

extractor = Extractor(
    temperatures_flat, noise_level_flat,
    names=['foreground', 'signal'],
    training_sets=[fg_training_flat, signal_training_set],
    dimensions=dimension,
    compiled_quantity=quantity,
    quantity_to_minimize='BPIC',
    expanders=expander_list,
    mean_translation=True
)
extractor.run()
```

---

## Common Patterns and Idioms

### HDF5 Persistence
Most pylinex objects support save/load via HDF5:
```python
import h5py

# Saving
with h5py.File('results.hdf5', 'w') as f:
    group = f.create_group('my_basis')
    basis.fill_hdf5_group(group)

# Loading
with h5py.File('results.hdf5', 'r') as f:
    basis = Basis.load_from_hdf5_group(f['my_basis'])
```

### Error Specification
Error can be:
- 1D array: `(nchannels,)` — diagonal noise standard deviations
- `SparseSquareBlockDiagonalMatrix` — block-diagonal covariance

```python
from distpy import SparseSquareBlockDiagonalMatrix
covariance = SparseSquareBlockDiagonalMatrix([block1, block2, ...])
```

### Multi-component Extraction Pattern
```python
from pylinex import TrainedBasis, BasisSum, Fitter, Extractor
import numpy as np

# Training sets (from simulations or files)
signal_training = np.load('signal_training.npy')      # (1000, 200)
foreground_training = np.load('fg_training.npy')       # (500, 200)

# Data
data = np.load('observed_spectrum.npy')                # (200,)
error = np.load('noise_levels.npy')                    # (200,)

# Quick fit at fixed truncation
signal_basis = TrainedBasis(signal_training, 5, error=error, mean_translation=True)
fg_basis = TrainedBasis(foreground_training, 3, error=error, mean_translation=True)
basis_sum = BasisSum(['signal', 'foreground'], [signal_basis, fg_basis])

fitter = Fitter(basis_sum, data, error=error)
print(fitter.channel_RMS)          # residual RMS
print(fitter.fit_parameters[:5])   # signal coefficients

# Grid search over truncation levels
extractor = Extractor(
    data, error,
    names=['signal', 'foreground'],
    training_sets=[signal_training, foreground_training],
    dimensions=[{'signal': np.arange(1, 8), 'foreground': np.arange(1, 6)}],
    quantity_to_minimize='bias_score',
    mean_translation=True
)
extractor.run()
```

### MCMC Sampling Pattern
```python
from pylinex import (BasisModel, SumModel, GaussianLoglikelihood,
                     Sampler, NLFitter, BurnRule)
from distpy import (GaussianDistribution, GaussianJumpingDistribution,
                    DistributionSet, JumpingDistributionSet)

# Model
model = SumModel(['signal', 'foreground'],
                 [BasisModel(signal_basis), BasisModel(fg_basis)])

# Likelihood
loglike = GaussianLoglikelihood(data, error, model)

# Priors and proposals
nparams = model.num_parameters
guess_dset = DistributionSet()
# ... add distributions for each parameter ...

jump_dset = JumpingDistributionSet()
jump_dset.add_distribution(
    GaussianJumpingDistribution(proposal_covariance),
    model.parameters
)

# Run sampler
sampler = Sampler('chains.hdf5', num_walkers=32, loglikelihood=loglike,
                  jumping_distribution_set=jump_dset,
                  guess_distribution_set=guess_dset,
                  steps_per_checkpoint=50)
sampler.run(5000)
sampler.close()

# Analyze
with NLFitter('chains.hdf5', BurnRule(desired_fraction=0.5, thin=2)) as fitter:
    print(f"Acceptance: {fitter.acceptance_fraction:.3f}")
    print(f"Posterior mean: {fitter.mean}")
    fitter.triangle_plot()
```

---

## Important Notes for Code Generation

1. **Import style**: Users typically do `from pylinex import *` or import specific classes. The top-level `__init__.py` exports everything.

2. **Array shapes**: Training sets are always (ncurves, nchannels). Data is (nchannels,). Basis vectors are (k, nchannels).

3. **Error handling**: The `error` parameter throughout the library accepts either 1D noise levels (standard deviations) or a full covariance matrix via `SparseSquareBlockDiagonalMatrix`.

4. **HDF5 is the standard persistence format**. Use `h5py` for all save/load operations.

5. **BasisSum is central**: Almost everything flows through BasisSum — it's the bridge between training data and fitting.

6. **Priors in Fitter**: Keyword arguments like `signal_prior=GaussianDistribution(mean, cov)` where the key is `{basis_name}_prior`.

7. **CompiledQuantity**: When using MetaFitter or Extractor, define what to compute at each grid point using CompiledQuantity containing AttributeQuantity/FunctionQuantity objects.

8. **Sampler restart modes**: 'continue' appends to existing chain, 'reinitialize' starts fresh in same file, 'update' updates jumping distribution from chain history.

9. **Context managers**: NLFitter supports `with` statements for automatic file handle cleanup.

10. **Plotting**: Many classes have built-in plotting via matplotlib. Triangle plots, histograms, and residual plots are available directly from NLFitter and Fitter objects.
