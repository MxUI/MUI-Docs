# RBF Spatial Sampler

## Mathematical Formulation

The radial basis function (RBF) spatial sampler constructs an interpolation of
a scalar field $s(\mathbf{r})$ at any point $\mathbf{r}\in\mathbb{R}^d$ using
a weighted sum of basis functions centred at source points, augmented by a
polynomial term. This formulation enables high-order accurate, consistent, or
conservative black-box data transfer between non-matching discretisations.

The general RBF interpolation is

$$
s(\mathbf{r}) =
\sum_{i=1}^{N}
\alpha_i\,
\phi\!\left(\left\|\mathbf{r}-\mathbf{r}_i\right\|\right)
+ p(\mathbf{r}),
$$

where

* $\phi(\cdot)$ is the selected radial basis function;
* $\alpha_i$ are interpolation coefficients determined from the source data;
* $p(\mathbf{r})$ is a polynomial term that guarantees exact recovery of
  rigid-body translations and rotations.

This formulation follows the work of Rendall and Allen (2009).

## Matrix Formulation

Within MUI, the interpolation is written as

$$
s_i = \sum_{j=1}^{M} H_{i,j}\alpha_j + p_i,
$$

or, equivalently,

$$
\mathbf{s} = \mathbf{H}\vec{\alpha} + \mathbf{p}.
$$

The interpolation matrix is

$$
\mathbf{H} = \mathbf{A}_{as}\mathbf{C}_{ss}^{-1}.
$$

In conservative mode:

* $\mathbf{C}_{ss}$ represents the correlation matrix between local source
  points;
* $\mathbf{A}_{as}$ represents the correlation matrix between remote target
  points and local source points.

For consistent mode, the roles of these matrices are reversed.

Since $\mathbf{C}_{ss}$ is symmetric,

$$
\mathbf{H}^{T}
= \left(\mathbf{A}_{as}\mathbf{C}_{ss}^{-1}\right)^T
= \mathbf{C}_{ss}^{-1}\mathbf{A}_{as}^{T}.
$$

Therefore,

$$
\mathbf{C}_{ss}\mathbf{H}^{T} = \mathbf{A}_{as}^{T},
$$

or, in block-matrix form,

$$
\begin{bmatrix}
\mathbf{M} & \mathbf{P}^{T} \\
\mathbf{P} & \mathbf{0}
\end{bmatrix}
\mathbf{H}^{T}
=
\begin{bmatrix}
\mathbf{N} \\
\mathbf{P}
\end{bmatrix}.
$$

Here,

* $\mathbf{M}$ is the RBF correlation matrix between local source points;
* $\mathbf{N}$ is the RBF correlation matrix between remote target points and
  local source points;
* $\mathbf{P}$ is the polynomial constraint matrix.

MUI solves the matrix system using a Conjugate Gradient (CG) solver. The
superscript $T$ denotes matrix transpose.

## Supported Basis Functions

The RBF sampler supports Gaussian and Wendland compact basis functions. The
basis function is selected using the integer parameter `basisFunc_`:

| `basisFunc_` | Basis function |
|-------------:|----------------|
| 0 | Gaussian |
| 1 | Wendland C0 |
| 2 | Wendland C2 |
| 3 | Wendland C4 |
| 4 | Wendland C6 |

### Gaussian Basis

For the Gaussian basis, the shape parameter is determined from the
user-defined `cutOff` value according to

$$
\sigma = \frac{\sqrt{-\ln(\mathtt{cutOff})}}{r},
$$

where $r$ is the search radius. This choice ensures that

$$
\phi(r)=\mathtt{cutOff},
$$

making the Gaussian basis effectively compact within the search radius.

### Wendland Basis

For the Wendland basis functions (C0, C2, C4, and C6), the formulation follows
Wendland (2004) and Rendall and Allen (2009).

The Euclidean distance $d$ is normalised using the search radius,

$$
q=\frac{d}{r},
$$

and the corresponding compact-support kernel is evaluated using $q$.

## Partition of Unity Localisation

To improve scalability, MUI adopts a localised RBF formulation inspired by the
pointwise Partition of Unity (PoU) method proposed by Rendall and Allen (2009).

Instead of constructing one dense global interpolation matrix, the
interpolation domain is divided into a collection of overlapping local patches.
An independent RBF interpolation is performed within each patch, and the local
interpolants are blended using Partition of Unity weighting functions.

The resulting global interpolant is

$$
s_g(\mathbf{x}) = \sum_{p=1}^{P} w_p(\mathbf{x})s_p(\mathbf{x}),
$$

subject to

$$
\sum_{p=1}^{P} w_p(\mathbf{x}) = 1.
$$

Here,

* $s_p(\mathbf{x})$ is the local RBF interpolant associated with patch $p$;
* $w_p(\mathbf{x})$ is the Partition of Unity weighting function.

The weighting function is defined as

$$
w_p(\mathbf{x}) =
\frac{
\phi\!\left(\left\|\mathbf{x}-\mathbf{x}_p\right\|\right)
}{
\displaystyle\sum_{k=1}^{P}
\phi\!\left(\left\|\mathbf{x}-\mathbf{x}_k\right\|\right)
},
$$

where $\mathbf{x}_p$ denotes the centre of patch $p$. The `pouSize` constructor
parameter controls the number of points in each local partition. Its default is
50; setting it to `0` disables the partitioned approach.

Compared with a global RBF formulation, the PoU approach offers:

* significantly lower memory consumption;
* improved computational efficiency;
* better parallel scalability;
* the same interpolation framework for large distributed problems.

Consequently, the PoU-based RBF sampler is well suited to large-scale parallel
multiphysics coupling applications, where a global dense interpolation matrix
would otherwise become prohibitively expensive.
