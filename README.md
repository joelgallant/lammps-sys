# `lammps-sys`

Builds and generates Rust bindings for the C interface of LAMMPS, the [*Large-scale Atomic/Molecular Massively Parallel Simulator.*](http://lammps.sandia.gov/)

## Usage

`lammps-sys` is not currently on crates.io.  You can depend on it with a git dependency:

<!-- Please remember to update ALL TOML examples, not just this one! -->
```toml
[dependencies.lammps-sys]
tag = "v0.5.0"
git = "https://github.com/ExpHP/lammps-sys"
```

## Docs

<!-- NOTE: The cpp file has the doc comments, not the h file -->
See LAMMPS' [`library.cpp`].  This is the file that bindings will be generated to.

If you just want to see the rust signatures for the bindings, you can also generate those yourself:

```
git clone https://github.com/ExpHP/lammps-sys
cd lammps-sys
cargo doc --open
```

## Modes of operation

`lammps-sys` will first probe for a system `liblammps` using `pkg-config`, and, failing that, will build it from source. This behavior may also be configured through the `RUST_LAMMPS_SOURCE` environment variable.

See the following documents for additional information:

### Linking a system library

See [Linking a system LAMMPS library](doc/linking-a-system-library.md).

### Building from source

See [Automatically building LAMMPS from source](doc/building-from-source.md).

## Configuration

### Environment variables

The following environment variables are used by `lammps-sys` to control the build:

* **`RUST_LAMMPS_SOURCE`**
  * `RUST_LAMMPS_SOURCE=auto`:  Try to link a system library, else build from source. **(default)**
  * `RUST_LAMMPS_SOURCE=system`:  Always link the system lammps library (else report an error explaining why this failed)
  * `RUST_LAMMPS_SOURCE=build`:  Always build from source
* **`RUST_LAMMPS_MAKEFILE`** - A path to a LAMMPS Makefile on your local filesystem which will be used as a template by this crate when building from source.  See the files in [`src/MAKE`] of the LAMMPS source for examples.  If unset or left blank, the prepackaged makefile at `MAKE/Makefile.serial` will be used.

### Cargo features

#### `exceptions`

Enables the following API functions by ensuring that `LAMMPS_EXCEPTIONS` is defined:

```
lammps_has_error
lammps_get_last_error_message
```

The system library will be skipped if it was not built with the definition.

#### Optional packages

There are a number of cargo features named with the prefix `package-`.  These are in one-to-one correspondence with LAMMPS' optional features [documented here](https://lammps.sandia.gov/doc/Packages.html), and each feature `package-abc` corresponds directly to running the command `make yes-abc` prior to building LAMMPS.

Unfortunately, these features are insufficient to actually *use* some of the packages.  For instance, the POEMS package requires building and linking an additional library, which `lammps-sys` currently provides no way to achieve. (sorry!)

You should activate features for all of the packages used directly by your crate. Unfortunately, these currently only have an effect when building LAMMPS from source (see the cautionary discussion about packages in [Linking a system LAMMPS library](doc/linking-a-system-library.md)).

## Does it work?

For an easier time diagnosing building/linking issues, you can clone this repo and try running the `link-test` example.

```sh
$ git clone https://github.com/ExpHP/lammps-sys
$ cd lammps-sys
$ cargo run --example=link-test
LAMMPS (31 Mar 2017)
Total wall time: 0:00:00
```

## License

Like Lammps, `lammps-sys` is licensed under the (full) GNU GPL v3.0. Please see the file [`COPYING`](COPYING) for more details.

## Release notes

See [Release notes](relnotes.md).

## Citations

S. Plimpton, **Fast Parallel Algorithms for Short-Range Molecular Dynamics**, J Comp Phys, 117, 1-19 (1995)

<!-- These links should all be maintained to point to the version
     of lammps that is built by `lammps-sys`                      -->
[`src/MAKE`]: https://github.com/lammps/lammps/tree/patch_5Feb2018/src/MAKE
[`library.cpp`]: https://github.com/lammps/lammps/blob/patch_5Feb2018/src/library.cpp
[the lammps source]: https://github.com/lammps/lammps/tree/patch_5Feb2018
