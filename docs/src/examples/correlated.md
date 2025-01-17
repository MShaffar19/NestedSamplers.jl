# Correlated Gaussian

This example will explore a highly-correlated Gaussian using [`Models.CorrelatedGaussian`](@ref). This model uses a conjuage Gaussian prior, see the docstring for the mathematical definition.

## Setup

For this example, you'll need to add the following packages
```julia
julia>]add Distributions MCMCChains Measurements NestedSamplers StatsBase StatsPlots
```

```@setup correlated
using AbstractMCMC
using Random
AbstractMCMC.setprogress!(false)
Random.seed!(8452)
```

## Define model

```@example correlated
using NestedSamplers

# set up a 4-dimensional Gaussian
D = 4
model, logz = Models.CorrelatedGaussian(D)
nothing; # hide
```

let's take a look at a couple of parameters to see what the likelihood surface looks like

```@example correlated
using StatsPlots

θ1 = range(-1, 1, length=1000)
θ2 = range(-1, 1, length=1000)
logf = [model.loglike([t1, t2, 0, 0]) for t2 in θ2, t1 in θ1]
heatmap(
    θ1, θ2, exp.(logf),
    aspect_ratio=1,
    xlims=extrema(θ1),
    ylims=extrema(θ2),
    xlabel="θ1",
    ylabel="θ2"
)
```

## Sample

```@example correlated
using MCMCChains
using StatsBase
# using single Ellipsoid for bounds
# using Gibbs-style slicing for proposing new points
sampler = Nested(D, 50 * (D + 1); 
    bounds=Bounds.Ellipsoid,
    proposal=Proposals.Slice()
)
names = ["θ_$i" for i in 1:D]
chain, state = sample(model, sampler; dlogz=0.01, param_names=names)
# resample chain using statistical weights
chain_resampled = sample(chain, Weights(vec(chain[:weights])), length(chain));
nothing # hide
```

## Results

```@example correlated
chain_resampled
```

```@example correlated
corner(chain_resampled)
```

```@example correlated
using Measurements
logz_est = state.logz ± state.logzerr
diff = logz_est - logz
print("logz: ", logz, "\nestimate: ", logz_est, "\ndiff: ", diff)
```
