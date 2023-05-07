---
title: Scientific Software
subtitle: "_Open source scientific software I've authored._"
---

## Julia

### `GeneralAstrodynamics.jl`

I developed a Julia [package](https://github.com/cadojo/GeneralAstrodynamics.jl) alongside my graduate astrodynamics courses, and I've registered
this package in Julia's package registry! Right now, the package provides calculations relating
to conic orbits, and circular-restricted three-body orbits. I'm most proud of the package's 
Halo orbit and manifold solvers! At my highest aspiration, this package will one day serve a purpose 
similar to Python's excellent [`poliastro`](https://docs.poliastro.space/en/stable/).

### `AstrodynamicalModels.jl`

Common astrodynamical models are provided in another open-source Julia package I've written:
[`AstrodynamicalModels.jl`](https://github.com/cadojo/AstrodynamicalModels.jl).
This package currently includes restricted two-body, n-body, and circular restricted three-body spacecraft dynamics.
The package extends the excellent [`ModelingToolkit.jl`](https://mtk.sciml.ai); check that out if you're not familiar!
Playing around with the symbolic-numeric "bridge" that `ModelingToolkit` provides helped me to re-learn some basic 
dynamics concepts (e.g. "oh yeah, the hessian of the potential energy equation is equivalent to the Jacobian of the 
system's equations of motion!") Documentation is available [here](https://cadojo.github.io/AstrodynamicalModels.jl)!

### `ManipulatorKinematics.jl`

While this project is not yet open-source, it soon will be! I wrote this Julia package as a graduate student. It 
provides symbolic code generation for manipulator kinematics. To get this to work, I ended up reworking some 
simple C++, MATLAB, and Stan code generation capabilities in [`Symbolics.jl`](https://symbolics.juliasymbolics.org/stable/)!

### `PolynomialGTM.jl`

This project is an [unofficial implementation](https://github.com/cadojo/PolynomialGTM.jl) of publicly available 
polynomial approximations for NASA's Generic Transport Model aircraft. Check out this package if you want a sandbox 
dynamical model for a control theory project! I'm using this package as an example for a control theory [note-set](https://github.com/cadojo/controls) I'm writing.

### `HorizonsAPI.jl`

This project wraps the JPL Horizons REST API word-for-word! You can use this to do anything
the Horizons API allows; request planetary ephemeris, download close approach tables, and more.

### `HorizonsEphemeris.jl`

This projects wraps the word-for-word wrapper — `HorizonsAPI.jl` — with a simpler interface. 
It's also currently more limited! This package currently only supports downloading Cartesian
ephemeris. The `ephemeris` function returns CSV-formatted position and velocity data for any 
solar system body supported by JPL Horizons; the returned type supports the `Tables.jl` 
interface, so you can pass it right on to the `DataFrame` and related constructors without
issue!

### `SolarSystemSurrogates.jl`

This is yet unreleased, but I'm excited about it! I'm trying to train surrogate models on 
planetary ephemeris data, so you don't need to download ephemeris at all for casual applications.
Fingers crossed it works!