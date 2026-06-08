# Language Wrappers

Although the core MUI library is implemented in C++, it can also be accessed from several other programming languages through dedicated language wrappers.

MUI currently provides wrappers for:

* C
* Fortran
* Python

These wrappers allow applications written in different languages to participate in the same coupled simulation while using the same underlying MUI communication framework.

---

## Overview

The language wrappers expose MUI functionality through language-specific interfaces while preserving access to the core capabilities of the library.

The wrappers are designed to minimise the amount of language-specific code required to integrate MUI into an existing solver.

---

## C++ Core Library

The core of MUI is implemented in C++11 and forms the foundation of all language interfaces.

The library makes extensive use of template programming and modern C++ language features to provide a flexible and high-performance framework for multiphysics code coupling. This design allows MUI to support multiple dimensions, data types, interpolation schemes, and coupling algorithms while maintaining a compact and efficient implementation.

MUI v2.0 deliberately retains C++11 as its baseline language standard. This avoids dependencies on newer C++ features and ensures compatibility with a wide range of compilers and legacy scientific software environments. As a result, MUI can be integrated into existing HPC applications with minimal disruption to established build systems and workflows.

The language wrappers described below are built on top of this common C++ core and expose its functionality to applications written in other programming languages.

---

## C Wrapper

The C wrapper in MUI-v2.0 has undergone a comprehensive redesign to improve usability, dimensional flexibility, and compatibility with modern HPC environments. The previous implementation provided a minimal interface limited to $3-D$ configurations and a small subset of samplers and functions. The new wrapper offers a full-featured, modular interface supporting one-dimensional, two-dimensional and three-dimensional domains, with extensive coverage of samplers, algorithms, and communication utilities.

The C wrapper in MUI-v2.0 serves as a bridge between C-based applications and the underlying C++ core of the MUI library. It achieves this by exposing a set of C-compatible functions that internally invoke the corresponding C++ classes and methods. Each wrapper module includes header files defining C-style structs and function prototypes, while the implementation files use extern "C" linkage to ensure compatibility with C compilers. These functions wrap around C++ constructs such as `uniface`, `sampler`, and `algorithm` objects, allowing C programs to instantiate, manipulate, and query MUI interfaces without directly interacting with C++ syntax or templates. Compilation is handled by compiling C source files with a C compiler and linking the resulting object files with the C++ wrapper library using a C++ compiler, ensuring seamless integration across language boundaries.

In the updated C wrapper in MUI-v2.0, `mui_c_wrapper_nd`, where `n`=`1`, `2`, `3`, provide tailored interfaces for one-dimensional, two-dimensional and three-dimensional domains, respectively. Each module defines its own `point_type, `uniface`, and `sampler` structures, allowing control over spatial and temporal resolution. This modular design enables users to select the appropriate dimensional interface for their application without modifying core logic.

Sampler support has been significantly expanded in the new wrapper. All built-in spatial and temporal samplers are now available in the C wrapper. These samplers are implemented across multiple precision types and are compatible with both single and multi-interface configurations, offering flexibility for diverse coupling scenarios.

The wrapper also includes general MPI communication utilities to support parallel execution. Functions such as 

```c
mui_mpi_split_by_app()
```

facilitate communicator management and enable hybrid parallel applications to initialise and manage MPI communicators more effectively. The build and testing infrastructure has been modernised to support both shared and static library generation (`libMUI_C_wrapper.so`, `libMUI_C_wrapper.a`) using CMake-compatible flags. Two unit tests are provided to validate functionality. The unit test `unit_test_single.c` demonstrates the creation and use of a single interface, while `unit_test_multi.c` showcases multi-interface scenarios using the `create_uniface<config>()` support function.

The wrapper is located in

```text
MUI/wrappers/C
```

Examples demonstrating the C wrapper are available in:

```text
MUI-demo/10-wrappers/10-0-C
```

---

## Fortran Wrapper

The Fortran wrapper in MUI-v2.0 provides a robust interface for Fortran applications to interact with the C++ core of the MUI library. It achieves this by leveraging the `ISO_C_BINDING` module introduced in Fortran 2003, which allows seamless interoperability between Fortran and C. Internally, the wrapper uses a set of C++ functions declared with extern "C" linkage, which are then exposed to Fortran through corresponding interface modules. These functions wrap around C++ constructs such as `uniface`, `sampler`, and `algorithm` objects, enabling Fortran programs to create, manipulate, and query MUI interfaces without directly dealing with C++ syntax or templates.

The updated Fortran wrapper introduces a modular structure with dedicated source files and modules for 1-D, 2-D, and 3-D domains (`mui_f_wrapper_1d`, `mui_f_wrapper_2d`, `mui_f_wrapper_3d`), as well as a general module (`mui_f_wrapper_general`) for dimension-independent functionality. Each dimensional module defines its own configuration structure (e.g., `mui_f_wrapper_1D`) specifying types such as `point_type, `time_type, and `iterator_type, which are used to instantiate template-based C++ interfaces tailored to the dimensionality and precision requirements of the application.

Significant improvements have been made over the previous version. The new wrapper supports both single and multi-interface configurations, with support functions like

```fortran
create_uniface_multi_1t_f()
```

enabling dynamic creation of multiple interfaces from Fortran. It also includes a comprehensive set of built-in spatial and temporal samplers across multiple precision types and full compatibility with the new `iterator_type` for sub-iteration support.

The build system has been modernised to support both shared and static library generation (`libMUI_Fortran_wrapper.so`, `libMUI_Fortran_wrapper.a`) using CMake-compatible flags. Compilation requires a Fortran 2003-compliant compiler and C++11 support. The wrapper also includes MPI communication utilities such as 

```fortran
mui_mpi_split_by_app_f()
```

to facilitate communicator management in hybrid parallel applications. Unit tests have been expanded to include both single and multi-interface scenarios (`unit_test.f90`, `unit_test_multi.f90`), demonstrating the wrapper's capabilities in realistic coupling setups.

The source code of the Fortran wrapper can be found in 

```text
MUI/wrappers/Fortran
```

Examples demonstrating the Fortran wrapper are available in:

```text
MUI-demo/10-wrappers/10-1-Fortran
```

---

## Python Wrapper

The Python wrapper is a new addition introduced in MUI-v2.0, and was not available in the earlier version. This wrapper provides a Python API that exposes the full functionality of the MUI-v2.0 library, enabling Python-based codebases and simulation frameworks to integrate MUI with minimal effort. Designed to support scripting workflows and rapid prototyping, the Python wrapper significantly lowers the barrier for using MUI in modern scientific computing environments.

This wrapper is implemented using pybind11, a lightweight header-only library that enables seamless interoperability between C++ and Python. The `mui4py` module exposes the essential MUI functionality, including geometry abstractions, spatial and temporal samplers, coupling algorithms, and the unified interface (`uniface`) implementations across multiple dimensionalities and numerical precisions. By leveraging pybind11's automatic type conversion and memory management capabilities, the wrapper maintains high-performance characteristics while providing interfaces for Python integration that are intuitive for domain scientists and engineers.

The development of the Python wrapper using pybind11 was motivated by the need to democratise access to MUI's sophisticated multiphysics and multiscale coupling capabilities across diverse computational workflows. pybind11 was selected as the binding mechanism because it offers minimal computational overhead, requires no external dependencies beyond Python headers, and produces highly readable, maintainable C++ binding code. Notably, since MUI is itself a header-only library, the binding layer leverages template specialisation across multiple configurations (varying dimensions and data types) to instantiate the necessary class and function definitions at compile time. This design allows the pybind11 wrapper to maintain full performance parity with the underlying C++ library while avoiding binary compatibility issues that would arise from traditional shared library distributions. The wrapper structure modularises different MUI components into separate C++ compilation units, such as geometry, samplers, algorithms, and uniface instantiations, facilitating code organisation and reducing compilation times while maintaining transparent access to the underlying high-performance C++ implementation.

The Python wrapper package builds with setuptools and CMake, discovering the correct pybind11 CMake package at build time and emitting the wheel and/or shared object into the Python package directory. The build script detects the platform’s Python/NumPy configuration, locates the user’s MPI and mpi4py headers, and sets the appropriate preprocessor defines. Runtime dependencies are minimal, which are mpi4py and NumPy for MUI-v2.0. The package pybind11 is a compile–time requirement only. This approach keeps the wrapper portable across HPC systems. The wrapper exports utilities 

```python
get_mpi_version()
get_compiler_version()
get_compiler_config()
```

to aid in reproducibility.

The wrapper is located in 

```text
MUI/wrappers/Python
```

Examples demonstrating the Python wrapper are available in:

```text
MUI-demo/10-wrappers/10-2-Python
```

---

## Wrapper Demonstrations

Examples demonstrating the language wrappers are provided in:

```text
MUI-demo/10-wrappers
```

The demonstration suite contains self-contained examples showing how MUI can be integrated into applications written in different programming languages.

These examples include build scripts and execution scripts to simplify testing and deployment.
