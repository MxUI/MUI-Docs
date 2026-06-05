# Language Wrappers

Although the core MUI library is implemented in C++, it can also be accessed from several other programming languages through dedicated language wrappers.

MUI currently provides wrappers for:

* C
* Fortran
* Python

These wrappers allow applications written in different languages to participate in the same coupled simulation while using the same underlying MUI communication framework.

---

# Overview

The language wrappers expose MUI functionality through language-specific interfaces while preserving access to the core capabilities of the library.

The wrappers are designed to minimise the amount of language-specific code required to integrate MUI into an existing solver.

---

# Wrapper Demonstrations

Examples demonstrating the language wrappers are provided in:

```text
MUI-demo/10-wrappers
```

The demonstration suite contains self-contained examples showing how MUI can be integrated into applications written in different programming languages.

These examples include build scripts and execution scripts to simplify testing and deployment.
