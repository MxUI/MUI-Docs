# Installation

This section describes how to build and install MUI using CMake, and how to use the installed package from a downstream solver.

Although the C++ core of MUI is header-only, the project provides a CMake-based installation workflow. This installs the required header files and CMake package configuration files, and can optionally build the C, Fortran, and Python wrappers.

## Requirements

To build and use MUI, you need:

* A C++11-compatible compiler
* MPI
* CMake

Additional compiler or interpreter support is required if you enable the C, Fortran, or Python wrappers.

## Build and Install via CMake

A typical local installation with all wrappers enabled is shown below.

```bash
git clone https://github.com/MxUI/MUI.git
cd MUI
mkdir build
cd build

cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=$HOME/.local \
      -DC_WRAPPER=ON \
      -DFORTRAN_WRAPPER=ON \
      -DPYTHON_WRAPPER=ON ..

cmake --build . --target install -j
```

The most relevant CMake options are:

| Option                 | Description                                                              |
| ---------------------- | -------------------------------------------------------------------------|
| `CMAKE_INSTALL_PREFIX` | Installation location with default of system path.                       |
| `CMAKE_BUILD_TYPE`     | Set the compilation type for wrappers, for example `Release` or `Debug`. |
| `C_WRAPPER`            | Enables or disables the C wrapper build.                                 |
| `FORTRAN_WRAPPER`      | Enables or disables the Fortran wrapper build.                           |
| `PYTHON_WRAPPER`       | Enables or disables the Python wrapper build.                            |

For example, to install MUI without wrappers:

```bash
cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=$HOME/.local \
      -DC_WRAPPER=OFF \
      -DFORTRAN_WRAPPER=OFF \
      -DPYTHON_WRAPPER=OFF ..
```

## Using the Installed Package in a Solver

After installation, downstream CMake projects can locate MUI using `find_package`.

A minimal `CMakeLists.txt` example is:

```cmake
find_package(MPI REQUIRED)
find_package(MUI REQUIRED)

add_executable(my_solver main.cpp)

target_link_libraries(my_solver PRIVATE MUI::MUI MPI::MPI_CXX)
target_include_directories(my_solver PRIVATE ${MUI_INCLUDE_DIRS})
```

If CMake cannot find the installed MUI package automatically, provide the installation prefix through `CMAKE_PREFIX_PATH`:

```bash
cmake -DCMAKE_PREFIX_PATH=/path/to/your/MUI/install ..
```

For example, if MUI was installed to `$HOME/.local`, use:

```bash
cmake -DCMAKE_PREFIX_PATH=$HOME/.local ..
```

## Checking MUI Detection in a Solver

When MUI is correctly detected through the exported `MUI::MUI` target, the following compile-time definitions are available:

* `HAVE_MUI_DETECTED`
* `MUI_VERSION`

These can be used to verify that the solver has been correctly linked against MUI:

```cpp
#ifdef HAVE_MUI_DETECTED
  std::cout << "HAVE_MUI_DETECTED: yes\n";

  if (MUI_VERSION >= 20000)
    std::cout << "MUI_VERSION = " << MUI_VERSION
              << ", which is 20,000 or larger\n";
  else
    std::cout << "MUI_VERSION = " << MUI_VERSION
              << ", which is smaller than 20,000\n";
#else
  std::cout << "HAVE_MUI_DETECTED: no\n";
#endif
```

This provides a simple validation mechanism for coupled solvers and confirms that the correct version of MUI installation is being used.

## Notes

MUI is a header-only library for C++ applications. Therefore, the core C++ functionality is able to be used without pre-compilation. Build the core C++ code through CMake can expose headers and CMake package files for downstream solvers. The C, Fortran, and Python wrappers must be built if those language interfaces are required.
