# `lammps-sys`

Builds and generates Rust bindings for the C interface of LAMMPS, the [*Large-scale Atomic/Molecular Massively Parallel Simulator.*](http://lammps.sandia.gov/)

## Usage

`lammps-sys` is not currently on crates.io.  You can depend on it with a git dependency:

<!-- Please remember to update ALL TOML examples, not just this one! -->
```toml
[dependencies.lammps-sys]
tag = "v0.4.0"
git = "https://github.com/ExpHP/lammps-sys"
```

## Docs

<!-- NOTE: The cpp file has the doc comments, not the h file -->
See LAMMPS' [`library.cpp`].  This is the file that bindings will be generated to.

If you just want to see the rust signatures for the bindings, you can also generate those yourself:

```
git clone https://github.com/ExpHP/lammps-sys
cd lammps-sys
cargo doc
chromium target/doc/lammps_sys/index.html
```

## Linking

As of version v0.4, *only static linking is supported.*

LAMMPS is automatically downloaded and built from source.  The default settings hopefully work out-of-the-box on most systems, though they might not be fast.

## Does it work?

For an easier time diagnosing building/linking issues, you can clone this repo and try running the `link-test` example.

```sh
$ git clone https://github.com/ExpHP/lammps-sys
$ cd lammps-sys
$ cargo run --example=link-test
LAMMPS (31 Mar 2017)
Total wall time: 0:00:00
```

Be sure to try this using the environment variables and `--features` that you plan to enable in your own project.

## Configuration

### Environment variables

The following environment variables are used by `lammps-sys` to control the build:

* **`RUST_LAMMPS_MAKEFILE`** - A path to a LAMMPS Makefile on your local filesystem which will be used as a template by this crate.  See the files in [`src/MAKE`] of the LAMMPS source for examples.  If unset or left blank, the prepackaged makefile at `MAKE/Makefile.serial` will be used.

### Cargo features

#### Preprocessor Flags

The following flags add functions to the C API, and can be enabled through features.

| Feature | Effect | Package docs |
| ------- | ------ | ----- |
| **`exceptions`** | `-DLAMMPS_EXCEPTIONS` | Makes Lammps report errors through new API functions. **Without this feature, LAMMPS will abort the process on error.** |
| `bigbig`  | `-DLAMMPS_BIGBIG` | Adds support for ridiculously large structures, and adds some related functions to the C bindings. |

**Bolded** features are enabled by default.

#### Packages

Lammps' packages are configured through cargo features, which cause `lammps-sys` to run the appropriate makefile target before building Lammps. Only a few packages are currently supported; I only added what I needed. PRs to add features for more packages are welcome.

| Feature | Effect | Package docs |
| ------- | ------ | ----- |
| `user-misc` | Runs `make yes-user-misc` | [USER-MISC package](http://lammps.sandia.gov/doc/Section_packages.html#user-misc-package) |
| `user-omp`  | Runs `make yes-user-omp` | [USER-OMP package](http://lammps.sandia.gov/doc/accelerate_omp.html) |

In theory, using cargo features to control packages allows a library that depends on `lammps-sys` to enable a few specific packages it requires without having to dictate the entire build. (Though in practice I rather doubt any library will ever depend on `lammps-sys`!)

## How-to

### Enabling OpenMP

Enabling OpenMP will require you to supply your own Makefile.

* If you are not familiar with the process of building LAMMPS, clone [the lammps source] and follow their instructions to learn how to compile an executable.
* Compile a LAMMPS executable with OpenMP support. This will require you to browse around their prepackaged makefiles and possibly make one of your own. You will know you have succeeded when the lammps binary accepts the command `"package omp 0"` and does not emit any warnings or errors.
* Set the environment variable `RUST_LAMMPS_MAKEFILE` to an absolute path to the makefile.

#### Supplying `-fopenmp` at linking

If you get errors during the linking stage about undefined symbols from `omp_`, you may try adding the following to your `~/.cargo/config` (or to a `.cargo/config` in your own crate's directory):

```
# For building lammps-sys with openmp.
# Not sure how this impacts other crates...

[target.x86_64-unknown-linux-gnu]
rustflags = ["-Clink-args=-fopenmp"]
```

(you can obtain the correct target triple for your machine by running `rustc --version -v`)

If this sounds like terrible advice, that's because it probably is!  Unfortunately, it does not seem to be possible to set this flag from within a cargo build script, and I do not know of a better solution at this time.

### Using a specific/modified version of lammps

This is not currently supported by `lammps-sys` in any fashion.

Prior versions of `lammps-sys` used the dynamic linking model, which made this possible. However, it required a bunch of manual setup and ultimately there is no way to ensure that the bindings generated by bindgen accurately reflect the library they are linking against.

<!-- DO NOT UPDATE THIS VERSION NUMBER! -->
<!-- It should remain at 0.3.0, the last version with dynamic linking. -->
If that does not deter you, however, feel free to check it out: [`lammps-sys v0.3.0`](https://github.com/ExpHP/lammps-sys/tree/v0.3.0)

## [License](COPYING)

Like Lammps, `lammps-sys` is licensed under the (full) GNU GPL v3.0. Please see the file `COPYING` for more details.

## [Release notes](relnotes.md)

## Citations

S. Plimpton, **Fast Parallel Algorithms for Short-Range Molecular Dynamics**, J Comp Phys, 117, 1-19 (1995)

<!-- These links should all be maintained to point to the version
     of lammps that is built by `lammps-sys`                      -->
[`src/MAKE`]: https://github.com/lammps/lammps/tree/patch_5Feb2018/src/MAKE
[`library.cpp`]: https://github.com/lammps/lammps/blob/patch_5Feb2018/src/library.cpp
[the lammps source]: https://github.com/lammps/lammps/tree/patch_5Feb2018
