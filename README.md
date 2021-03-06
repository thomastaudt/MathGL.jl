
# Contents

- [Requirements ](#requirements)
- [Usage and Examples ](#usage-and-examples)
    - [The C Interface ](#the-c-interface)
    - [The Higher Level Julian Interface ](#the-higher-level-julian-interface)
    - [Abusing The Matrix Syntax: Imitating Mgl Scripts in Julia ](#abusing-the-matrix-syntax-imitating-mgl-scripts-in-julia)
- [Examples](#examples)
- [Installing](#installing)
- [Changes to MathGL](#changes-to-mathgl)
- [TODO](#todo)

<!-- end toc -->

# MathGL.jl

![main example](/graphs/main_example.png?raw=true)

**This repository probably still has major issues and is a work in
progress**

MathGL.jl provides a wrapper for the feature-rich scientific visualization
library [MathGL](http://mathgl.sourceforge.net) written in C++. It relies
on the C interface of MathGL, wrapping it with julia to a convenient
degree.

MathGL.jl is currently under construction, still lacks a part of the
functions provided by MathGL and is probably not portable (only linux
tested). Most functions related to (non-gui-)plotting should be
implemented, though. 
The main priority of MathGL.jl is to make MathGL's capacity to create
graphs available in julia; implementing other parts of the library
(numerical routines, classes for storing numercial data, the pde solvers,
and fitting procedures) has lower priority. To still access these, the
low-level Capi must be used as of now (see [below](#the-c-interface)).

For more information about MathGL, see its
[documentation](http://mathgl.sourceforge.net/doc_en/Main.html)


### Requirements 
Besides the code of this repository and julia 4+, you will need a working
and up-to-date version of libmgl (version 2.3) in a folder that julia can find via
`find_library`. For additional gui features (which are *not* supported yet),
like qt, glut, etc. the corresponding libraries (e.g. libmgl-qt5) must
also be installed properly.


### Usage and Examples 
#### The C Interface 
The C functions provided by MathGL are callable via the Capi submodule of
MathGL. The Capi-module is more or less complete (some functions may be
missing, but most functionality is certainly implemented); however,
it is not comfortable to use. Documentation regarding the Capi (function
names and their description) may be found in the 
[MathGL documentation](http://mathgl.sourceforge.net/doc_en/Main.html).

An example: 
```julia
using MathGL.Capi
mgl = MathGL.Capi

gr  = mgl.create_graph(800, 500)
dat = mgl.create_data()
mgl.data_link(dat, 0.8*sin(linspace(-4pi, 4pi, 200)), 200, 1, 1)
mgl.label(gr, 'x', "x", 0, "")
mgl.label(gr, 'y', "y", 0, "")
mgl.box(gr)
mgl.axis(gr, "xyz", "", "")
mgl.axis_grid(gr, "xyz", "H|", "")
mgl.plot(gr, dat, "", "")
mgl.write_frame(gr, "sin_C.png", "")
```
![sin_C example](/graphs/sin_C.png?raw=true)
Note however, that the C interface only conducts limited type checks, so
expect segfaults when using the wrong argument types.

#### The Higher Level Julian Interface 
The type structure was (quite loosely) modeled after the C++ class
structure of MathGL, the most important type being the `Graph`. The
function names (e.g. `text`, `surf`, `plot`, `xtics`, ...) were -- whenever
possible -- chosen to be the corresponding commands of the MathGL scripting
language (see the [documentation](http://mathgl.sourceforge.net/doc_en/Main.html) for
the details). This was done for several reasons:
    
* The names of the mgl commands are very julian (e.g. few underscores,
  heavily overloaded)
* The C interface is tedious and the C++ interface does not really
  fit well
* An abuse of julia's matrix construction syntax makes it possible to
  write mgl script code directly in julia (with some slight
  alterations, see [below](#abusing-the-matrix-syntax-imitating-mgl-scripts-in-julia)).

There are, however, important functions that are not covered by the
scripting language, which is designed to handle one graph only.
Some of these deviations of function names can be found in the file
[changes.md](/changes.md), the most important ones are given
[below](#changes-to-mathgl).

An equivalent example to the one above in the julian interface:
```julia
using MathGL
mgl = MathGL

gr  = mgl.Graph(800, 500)
mgl.xlabel(gr, "x")
mgl.ylabel(gr, "y")
mgl.box(gr)
mgl.axis(gr)
mgl.grid(gr)
mgl.plot(gr, 0.8*sin(linspace(-4pi, 4pi, 200)))
mgl.write(gr, "sin_julia.png")
```

#### Abusing the Matrix Syntax: Imitating MathGL Scripts in Julia 
To make it short, the following code sample is perfectly equivalent to the
one above:
```julia
using MathGL

gr = MathGL.Graph(800, 500)
@mglplot gr [
    xlabel "x"
    ylabel "y"
    box
    axis
    grid
    plot 0.8*sin(linspace(-4pi, 4pi, 200))
    write "sin_mgl.png"
]
```
This is nice because the syntax given in the macro resembles the
MGL script syntax quite well. Of course there are deviations (e.g. always
the keyword `stl` is used, never `fnt` or `sch` or ...).

## Examples
Examples can be found in the folder [examples](/examples). Right now the only relevant
example is the one creating the picture at the top of this repository,
[here](/examples/main_example.jl).

## Installing
Simply clone the repository, and -- if so desired -- add the `src`
subdirectory to your `JULIA_LOAD_PATH`. Beware that at the moment I don't
know how to check if a 64bit version (using doubles) or a 32bit
version (using floats) of MathGL is installed, so the value of the
`typealias Float` in `src/capi.jl` must be adopted manually.

## Changes to MathGL
* Additional properties (like line color, dashed-ness, ...) are normally set in
  MGL script by keyword arguments like `pen`, `stl`, `fnt`, or `sch`. MathGL.jl
  always uses `stl` (for style).
* The C and C++ functions of MathGL sometimes expect lists of
  strings/labels by handing over one big string of the form
  "string1\nstring2\nstring3". In MathGL.jl these functions expect arrays
  ["string1", "string2", "string3"].
* Function names for which no correspondig MGL script function is named in
  the manual or for which the names are missleading were renamed:
  * `showwith` as function wrapping the MathGL C function `mgl_show_image`.
  * `get_width`, `get_height` are called `width`, `heigth`.
  * `axis` is used in MGL script for both (1) plotting the axes of the
     coordinate system and (2) setting the coordinate system (e.g. logarithmic
     axes). In MathGL.jl `axis` is used for (1), but `coords` is used for
     (2) instead, with keyword arguments X,Y,Z,C.
* Added `subgrid` function (which is just `grid` with `'!'` added.
 
## TODO
* Make all plotting functions accessible from julia
* Provide some more utility functions
* Write some documentation / tutorials for the usage of MathGL
* Implement the gui graph classes of MathGL
* Testing! Not much is tested, there are certainly tons of bugs yet
  undiscovered (but easy to resolve)
* Document the changes compared to MathGL
* Use more precise array subtypes
