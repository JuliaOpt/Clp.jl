# COIN-OR Linear Programming Interface (Clp.jl)

![](https://www.coin-or.org/wordpress/wp-content/uploads/2014/08/COINOR.png)

`Clp.jl` is an interface to the **[COIN-OR Linear
Programming](https://projects.coin-or.org/Clp)** solver. It provides a complete
interface to the low-level C API, as well as an implementation of the
solver-independent `MathProgBase` and `MathOptInterface` API's.   

*Note: This wrapper is maintained by the JuliaOpt community and is not a COIN-OR
project.*

[![Build Status](https://travis-ci.org/JuliaOpt/Clp.jl.svg?branch=master)](https://travis-ci.org/JuliaOpt/Clp.jl)
[![codecov](https://codecov.io/gh/JuliaOpt/Clp.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaOpt/Clp.jl)

[Clp]: https://projects.coin-or.org/Clp
[Cbc]: https://github.com/JuliaOpt/Cbc.jl

## Installation

The package is registered in `METADATA.jl` and so can be installed with `Pkg.add`.

```
julia> import Pkg; Pkg.add("Clp")
```

Clp.jl will use [BinaryProvider.jl](https://github.com/JuliaPackaging/BinaryProvider.jl) to automatically install the Clp binaries. This should work for both the official Julia binaries from `https://julialang.org/downloads/` and source-builds.

## Custom Installation

To install custom built Clp binaries set the environmental variable `JULIA_CLP_LIBRARY_PATH` and call `import Pkg; Pkg.build("Clp")`. For instance, if the libraries are installed in `/opt/lib`, then call
```julia
ENV["JULIA_CLP_LIBRARY_PATH"] = "/opt/lib"
import Pkg; Pkg.build("Clp")
```
If you do not want BinaryProvider to download the default binaries on install, set `JULIA_CLP_LIBRARY_PATH` before calling `import Pkg; Pkg.add("Clp")`.

To switch back to the default binaries clear `JULIA_CLP_LIBRARY_PATH` and call `import Pkg; Pkg.build("Clp")`.

### Using with **[JuMP]**
[JuMP]: https://github.com/JuliaOpt/JuMP.jl

Due to some restrictions in Clp's C api, the Clp's MathOptInterface wrapper does not support directly modifying a problem after it has been created, e.g., changing variable bounds or modifying constraints coefficients.

Therefore, we highly recommend that you use the `Clp.jl` package with higher-level package such as [JuMP.jl](https://github.com/JuliaOpt/JuMP.jl).
This can be done with following syntax:
```julia
using JuMP, Clp

model = Model(with_optimizer(Clp.Optimizer, LogLevel=1, Algorithm=4))
```

See the list of options below.

Furthermore, the following features are not supported:
* Querying the dual bound via `JuMP.objective_bound` (not in the C API)
* Setting a time limit (the C API behaves inconsistently, see [#65](https://github.com/JuliaOpt/Clp.jl/issues/65))
* Setting the number of threads used (not in the C API)
* Quadratic objective (not supported yet)
* Querying infeasibility certificates (bug in Clp)



### Using with **[MathProgBase]**


Clp provides a solver object that can be passed to ``linprog`` in MathProgBase (and used to create instances of the solver-independent ``AbstractMathProgModel`` type):

```julia
using Clp
using MathProgBase
linprog(..., ClpSolver(Option1=value1,Option2=value2,...))
```

See the list of options below, and the MathProgBase documentation for further information.

[MathProgBase]: https://github.com/JuliaOpt/MathProgBase.jl

### Using the C interface

The low-level C interface is available in the ``ClpCInterface`` submodule:
```julia
using Clp.ClpCInterface
```

Using this interface is only recommended for advanced users. The Julia API is essentially a thin wrapper around the interface exported by ``Clp/src/Clp_C_Interface.h``, which is documented in-line.


### Solver options

The following options are the most useful (and well documented):

| Parameter | Default value | Description |
|:----------|:--------------|:----------|
| `PrimalTolerance` | `1e-7` | Primal feasibility tolerance |
| `DualTolerance`   | `1e-7` | Dual feasibility tolerance |
| `DualObjectiveLimit` | `1e308` | When using dual simplex (where the objective is monotonically changing), terminate when the objective exceeds this limit |
| `MaximumIterations` | `2147483647` | Terminate after performing this number of simplex iterations |
| `MaximumSeconds` | `-1.0` | Terminate after this many seconds have passed. A negative value means no time limit |
| `LogLevel` | `1` | Set to 1, 2, 3, or 4 for increasing output. Set to `0` to disable output |
| `PresolveType` | `0` | Set to 1 to disable presolve |
| `SolveType` | `5` | Solution method: dual simplex (`0`), primal simplex (`1`), sprint (`2`) barrier with crossover (`3`), barrier without crossover (`4`), automatic (`5`) |
| `InfeasibleReturn` | `0` | Set to 1 to return as soon as the problem is found to be infeasible (by default, an infeasibility proof is computed as well) |
