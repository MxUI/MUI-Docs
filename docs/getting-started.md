# Getting Started

## Introduction

The Multiscale Universal Interface (MUI) is a lightweight library designed for coupling heterogeneous simulation codes in multiphysics and multiscale applications. It provides a unified data exchange framework that allows independently developed solvers to communicate efficiently regardless of implementation details.

MUI supports:

* Coupling of independent simulation codes
* Spatial and temporal data interpolation
* Non-conformal mesh transfers
* Parallel execution using MPI
* Coupling algorithms
* Multi-language support through C, Fortran, and Python wrappers

The core MUI library is implemented as a header-only C++ library and requires only MPI as an external dependency.

---

## Where to Obtain MUI Library

MUI and its companion repositories are publicly available through the MxUI GitHub organisation.

The MUI library, along with its companion repositories, is publicly available under either the Apache 2.0 or GPL-3 license, allowing users to select the option most suitable for their intended application. These resources are hosted under the MxUI GitHub organisation https://github.com/MxUI, which is actively maintained by the development team. 

### Core Library

The core MUI implementation contains the communication framework, interpolation algorithms, coupling utilities, and language wrappers. The core library source code is provided at

```bash
git clone https://github.com/MxUI/MUI.git
```
It is implemented as a header-only C++ library with a single dependency on MPI, making integration into existing codes straightforward and minimally intrusive.

### Demonstration Cases

A curated collection of demonstration cases is provided to illustrate different MUI capabilities, ranging from basic data transfer to advanced spatial and temporal interpolation, as well as iterative coupling algorithms.

```bash
git clone https://github.com/MxUI/MUI-demo.git
```

### Testing

A dedicated standalone parameterisable testing and benchmarking application is also available for scalability studies for high core counts.

```bash
git clone https://github.com/MxUI/MUI-Testing.git
```

---

## Software Requirements

To use MUI with C++ applications, the following software is required:

| Software                  | Purpose                             |
| ------------------------- | ----------------------------------- |
| MPI                       | Parallel communication              |
| C++11 compatible compiler | Building coupled applications       |
| CMake                     | Installation and package management |

Additional requirements may apply when building:

* C wrappers
* Fortran wrappers
* Python wrappers

---

## MUI Architecture

MUI acts as an intermediary communication layer between independent simulation codes.

```text
+-----------+       +-----------+
| Solver A  | <---> |    MUI    |
+-----------+       +-----------+
                         ^
                         |
                         v
                    +-----------+
                    | Solver B  |
                    +-----------+
                         ^
                         |
                         v
+-----------+       +-----------+
| Solver C  | <---> |    MUI    |
+-----------+       +-----------+
     ^
     |
     v
    ...
```

Each participating solver communicates through one or more MUI interfaces. Data is exchanged using a push-fetch mechanism and may be spatially and temporally interpolated during transfer.

