# MuRiT
Copyright © 2022 Michael Bleher, Maximilian Neumann, Lukas Hahn, Andreas Ott

## Description
`MuRiT` is a standalone program that provides Vietoris-Rips transformations for multifiltered flag complexes.

It is build with the main goal of providing a `Ripser` add-on to calculate persistent homology of flag complexes along any totally ordered subfiltration of interest.
Thus, `MuRiT` allows an easy exploration of multi-persistence properties for filtered metric spaces, e.g. timeseries data.


The main features of `MuRiT` are

- easy exploration of multi-persistence for filtered metric spaces
- optimized for use as `Ripser` add-on (integrated call to local instance of `Ripser`)
- fully parallelized and memory-efficient creation of the distance matrix of a Vietoris-Rips transformation
- standalone exectubales for all major platforms
- open source go code


## Setup
### Installation
We provide executables for major platforms and architectures as releases, no installation is required for those.
Just download the executable that fits your setup.

If your platform and architecture is not provided, please see below.

### Build and Modify
`MuRiT` is written in go, so it can be easily modified and run directly from source.
For this you only need [a recent go installation](https://go.dev/) and a copy of this repository.

To run `MuRiT` from source use
```
go run main.go [--options]
```

For help in building your own executables, [see here](https://go.dev/doc/tutorial/compile-install).

### Contribute
We envision a broad interaction with the research community.
If you have a special use-case for `MuRiT` that isn't covered by the current implementation or have an idea for additional features, get in touch or submit your pull requests.


## Using `MuRiT`
`MuRiT` expects as input

- a comma-separated lower-triangular distance matrix,
- a pointwise annotation file,
- and a one-dimensional subfiltration $(VR_0, i_0, j_0, ...)-- ... --(VR_n, i_n, j_n, ...)$.

It's output is

- either the distance matrix of a Vietoris-Rips Transformation of the flag complex
- or the persistent homology of the flag complex along the chosen subfiltration.

A call to `MuRiT` looks as follows

```
murit --dist [filename] --pt_fltr [filename] --sub_fltr (VR_0, i_0, j_0, ...)-- ... --(VR_n, i_n, j_n, ...) [--options]
```

*Note: MuRiT relies on the standard partial order on $\mathbb{R}^n$ for some $n$. Make sure you have translated*

### List of Command Line Arguments

##### `--dist`

file name of lower-triangular distance matrix.


##### `--pt_fltr`

file name of pointwise filtration annotation.

  file content: on row 'i' a comma-separated list of minimal filtration values for data point 'i'.
  example file:


    (0,0,1)     // minimum of point 1
    (1,1,1)     // minimum of point 2
    ...

##### `--sub_fltr`

  command line input of sub-filtration along which to compute 1d persistence.

  example:
    (VR_0, i_0, j_0, k_0,...)-- ... --(VR_n, i_n, j_n, k_n,...)

##### `--threads`
number of threads (default: runtime.NumCPU())
##### `--verbose`
Show status messages (default: false)
##### `--help`
Show help message
##### `--ripser`
run ripser on auxiliary distance matrix (default: false).
Requires a local ripser installation in PATH

Further Ripser options
- `--dim` compute persistent homology up to dimension k (default: 1).
- `--threshold` compute persistent homology up to threshold t (in auxiliary distance matrix, default: enclosing radius).
- `--modulus` compute homology with coefficients in the prime field Z/pZ (default: 2).
- `--ratio` only show persistence pairs with death/birth ratio > r


## Example

_The distance matrix and filtration file for the following example is provided in `examples/diamonds.*`_

Consider the metric space of a diamond consisting of four points with side length $1$ and diagonal length $2$.
Assume that for each point there is additional information about time $t$ at which the point entered the data set (say in seconds), and some threshold height $h$ above which it could be observed.

The following list of filtration tuples $(t, h)$ for each of the four points is provided in `examples/diamonds.fltr`.
```
(0,0)
(1.2,0)
(1.6,1)
(3.1,1)
```
This means that the first point was observed from the beginning and already at lowest height, while the second point only appears after 1.2 seconds and also at lowest height, et cetera.

Running `MuRiT` on this data produces the distance matrix of a Vietoris-Rips transformation
```
$ murit --dist examples/diamond.dist --pt_fltr examples/diamond.fltr --verbose
Read pointwise annotation file
Build subfiltration
[[0 0 0] [1 0 0] [1 1.2 0] [1 1.6 1] [1 2 1]]
Build auxiliary Distance Matrix
3
4,5
5,5,5
```
_Note: If no subfiltration is specified, `MuRiT` will build some auxiliary initial subfiltration from information in the filtration annotation. We made use of this behaviour in this example._

If we add the `--ripser` flag, the distance matrix is automatically handed over to `Ripser` to calculate persistent homology along the one-dimensional subfiltration.
```
$ murit --dist examples/diamond.dist --pt_fltr examples/diamond.fltr --verbose --ripser
Read pointwise annotation file
Build subfiltration
[[0 0 0] [1 0 0] [1 1.2 0] [1 1.6 1] [1 2 1]]
Build auxiliary Distance Matrix
---
Run Ripser
value range: [3,5]
distance matrix with 4 points, using threshold at enclosing radius 5
persistent homology intervals in dim 1:

```
Note that we do not find non-trivial homology in this example.
This makes sense, because the last point enters the filtration only at the last step in the one-dimensional subfiltration.
Only from that point on we might expect to see the cycle around the diamond.

Adding two further filtration steps $[2,2,1]--[2.1,2,1]$ at higher Vietoris-Rips parameters to the automatically generated subfiltration, we get:
```
$ murit --dist examples/diamond.dist --pt_fltr examples/diamond.fltr --verbose --ripser --sub_fltr [0,0,0]--[1,0,0]--[1,1,0]--[1,2,0]--[1,2,1]--[2,2,1]--[2.1,2,1]
Read pointwise annotation file
Read subfiltration
[[0 0 0] [1 0 0] [1 1 0] [1 2 0] [1 2 1] [2 2 1] [2.1 2 1]]
Build auxiliary Distance Matrix
---
Run Ripser
value range: [4,6]
distance matrix with 4 points, using threshold at enclosing radius 6
persistent homology intervals in dim 1:
 [ [1 2 1] , [2 2 1] ):
{[0,2] (5), [1,3] (5), [2,3] (5), [0,1] (4)}
{[0,2] (5), [1,3] (5), [2,3] (5), [0,1] (4)}
```
Indeed we find that there is a homology class in dimension one, which is born at filtration value $[1,2,1]$, i.e. once the fourth point has been observed.
The homology class dies at filtration value $[2,2,1]$, which is when the diagonals are added to the Vietoris-Rips complex.


_Note: By using the `--ripser` flag, the result is automatically converted back to the subfiltration values._


## Citing

Maximilian Neumann, Michael Bleher, Lukas Hahn, Samuel Braun,
Holger Obermaier, Mehmet Soysal, René Caspart, and Andreas Ott (2022).
" MuRiT: efficient computation of induced persistence barcodes of multi-filtered flag complexes via Vietoris-Rips transformations". *(to appear)*
