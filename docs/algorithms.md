# Coupling Algorithms

MUI provides coupling algorithm modules that can be applied directly at the data exchange layer. These algorithms accelerate convergence in partitioned multiphysics simulations without requiring intrusive modifications to the coupled solvers.

The algorithm implementations are demonstrated in:

```text
MUI-demo/09-algorithms
```

Currently, MUI provides examples of:

* Fixed Relaxation
* Aitken Dynamic Relaxation

These algorithms can be used in coupled simulations and support restart functionality and dynamic point sets.

---

## Overview

In partitioned multiphysics simulations, different solvers exchange interface data iteratively until convergence is achieved.

A simple fixed-point iteration may converge slowly or become unstable. Relaxation techniques improve robustness and often significantly reduce the number of coupling iterations required.

MUI implements these algorithms directly within the data exchange framework, allowing users to add relaxation behaviour with minimal changes to solver code.

---

## Aitken Dynamic Relaxation

One of the most widely used acceleration techniques for partitioned coupling is Aitken's dynamic relaxation method.

The implementation is demonstrated in:

```text
MUI-demo/09-algorithms/9-1-Aitken
```

A shortened example is shown below.

```cpp
int main(int argc, char ** argv) {

  using namespace mui;

  MPI_Comm world = mui::mpi_split_by_app();

  std::vector<std::pair<mui::point1d, double>> ptsVluInit;

  algo_aitken1d aitken(0.01,
                       1.0,
                       world,
                       ptsVluInit,
                       0.023969);

  for (int t = 1; t <= 10; ++t) {

    for (int iter = 1; iter <= 100; ++iter) {

      interface.push("u", 4, u[4]);

      interface.commit(t, iter);

      u[6] = interface.fetch("u0",
                             6 * H,
                             t,
                             iter,
                             s1,
                             s2,
                             aitken);

      printf("Under relaxation factor = %f\n",
             aitken.get_under_relaxation_factor(t, iter));

      printf("Residual L2 norm = %f\n",
             aitken.get_residual_L2_Norm(t, iter));
    }
  }

  return 0;
}
```

---

## Creating an Aitken Algorithm Instance

To use the algorithm, create an instance before entering the coupling loop.

```cpp
algo_aitken1d aitken(0.01,
                     1.0,
                     world,
                     ptsVluInit,
                     0.023969);
```

The constructor accepts:

| Parameter                     | Description                                    |
| ----------------------------- | ---------------------------------------------- |
| `under_relaxation_factor`     | Initial relaxation factor                      |
| `under_relaxation_factor_max` | Maximum allowed relaxation factor              |
| `local_comm`                  | MPI communicator used for residual evaluation  |
| `pts_vlu_init`                | Optional initial point-value pairs for restart |
| `res_l2_norm_nm1`             | Previous residual norm used for restart        |

This design allows both fresh and restarted coupling runs.

---

## Applying the Algorithm

Once constructed, the algorithm object is passed directly to the `fetch()` operation.

```cpp
u[6] = interface.fetch("u0",
                       6 * H,
                       t,
                       iter,
                       s1,
                       s2,
                       aitken);
```

MUI automatically applies the relaxation procedure during data retrieval.

---

## Monitoring Convergence

The current relaxation factor can be queried at runtime.

```cpp
aitken.get_under_relaxation_factor(t, iter);
```

Similarly, the current residual norm can be obtained using:

```cpp
aitken.get_residual_L2_Norm(t, iter);
```

These diagnostics are useful for:

* convergence monitoring
* debugging
* performance assessment
* automated stopping criteria

For example:

```cpp
std::cout
    << aitken.get_under_relaxation_factor(t, iter)
    << std::endl;

std::cout
    << aitken.get_residual_L2_Norm(t, iter)
    << std::endl;
```

---

## Fixed Relaxation

MUI also provides fixed-relaxation algorithms.

The examples can be found in:

```text
MUI-demo/09-algorithms/9-0-fixedRelaxation
```

Unlike Aitken's method, the relaxation factor remains constant throughout the coupling process.

Fixed relaxation is often useful when:

* convergence behaviour is well understood
* robustness is more important than convergence speed
* a simple relaxation strategy is sufficient

For strongly coupled problems, Aitken relaxation generally provides faster convergence because the relaxation factor is updated automatically based on the residual history.

---

## Restart Capability

Large multiphysics simulations may require restart functionality after:

* job interruptions
* queue time limits
* solver failures
* planned checkpointing

MUI coupling algorithms support restarting from a previously saved state.

Users can initialise the algorithm using previously stored point-value pairs:

```cpp
std::vector<std::pair<point_type, REAL>> pts_vlu_init;
```

and provide the previous residual norm during construction.

```cpp
algo_aitken1d aitken(urf,
                     urf_max,
                     world,
                     pts_vlu_init,
                     previous_residual);
```

---

## Dynamic Point Handling

Many multiphysics simulations involve:

* moving meshes
* adaptive mesh refinement
* changing interface topology

As a result, interface points may appear or disappear during execution.

MUI coupling algorithms automatically handle dynamic point sets without requiring additional user code.

Examples demonstrating this capability include:

```text
MUI-demo/09-algorithms/9-0-fixedRelaxation/9-0-2-dynamic_points
```

and

```text
MUI-demo/09-algorithms/9-1-Aitken/9-1-2-dynamic_points
```

The internal algorithm state is updated automatically when points are added or removed.

---

## Strong Coupling with Dual-Time Stepping

Strong coupling often requires several interface iterations within a single physical time step.

MUI supports this through extended versions of `commit()` and `fetch()` that accept both:

* physical time index
* coupling iteration index

For example:

```cpp
interface.commit(t, iter);
```

and

```cpp
interface.fetch("u0",
                point,
                t,
                iter,
                spatial_sampler,
                temporal_sampler,
                aitken);
```

This allows MUI to distinguish between:

* physical time advancement
* coupling sub-iterations

and enables the implementation of strongly coupled partitioned algorithms.
