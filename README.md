# Tarpaulin

[![Build Status](https://github.com/xd009642/tarpaulin/workflows/Build/badge.svg)](https://github.com/xd009642/tarpaulin/actions)
[![Latest Version](https://img.shields.io/crates/v/cargo-tarpaulin.svg)](https://crates.io/crates/cargo-tarpaulin)
[![License:MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/docker/automated/xd009642/tarpaulin.svg)](https://hub.docker.com/r/xd009642/tarpaulin/)
[![Developers Wiki](https://img.shields.io/badge/development-wiki-yellowgreen.svg)](https://github.com/xd009642/tarpaulin/wiki/Developers)

Tarpaulin is designed to be a code coverage reporting tool for the Cargo build
system, named for a waterproof cloth used to cover cargo on a ship. Currently,
tarpaulin provides working line coverage but is still in the early development
stage and therefore may contain some bugs. A lot of work has been done to get it
working on some example projects and smaller crates so please report anything
you find that's wrong. Also, check out our roadmap for planned features.

**Tarpaulin only supports x86_64 processors running Linux.** This is because
instrumenting breakpoints into executables and tracing their execution requires
processor and OS specific code. It is a goal when greater stability is reached
to add wider system support, however this is sufficient to run Tarpaulin on
popular CI tools like Travis.

It can also be run in Docker, which is useful for when you don't use Linux but
want to run it locally, e.g. during development. See below for how to do that.

Below is the help-text for a thorough explanation of the flags and features
available:

```
cargo-tarpaulin version: 0.10.2
Tool to analyse test coverage of cargo projects

USAGE:
    cargo tarpaulin [FLAGS] [OPTIONS] [-- <args>...]

FLAGS:
        --all                    Alias for --workspace (deprecated)
        --all-features           Build all available features
    -b, --branch                 Branch coverage: NOT IMPLEMENTED
        --count                  Counts the number of hits during coverage
        --debug                  Show debug output - this is used for diagnosing issues with tarpaulin
        --force-clean            Adds a clean stage to work around cargo bugs that may affect coverage results
    -f, --forward                Forwards unexpected signals to test. Tarpaulin will still take signals it is expecting.
        --frozen                 Do not update Cargo.lock or any caches
    -h, --help                   Prints help information
        --ignore-config          Ignore any project config files
        --ignore-panics          Ignore panic macros in tests
        --ignore-tests           Ignore lines of test functions when collecting coverage
    -i, --ignored                Run ignored tests as well
    -l, --line                   Line coverage
        --locked                 Do not update Cargo.lock
        --no-default-features    Do not include default features
        --no-run                 Compile tests but don't run coverage
        --offline                Run without accessing the network
        --release                Build in release mode.
    -V, --version                Prints version information
    -v, --verbose                Show extra output
        --workspace              Test all packages in the workspace

OPTIONS:
    -Z <FEATURES>...                 List of unstable nightly only flags
        --ciserver <SERVICE>         Name of service, supported services are:
                                     travis-ci, travis-pro, circle-ci, semaphore, jenkins and codeship.
                                     If you are interfacing with coveralls.io or another site you can also specify a
                                     name that they will recognise. Refer to their documentation for this.
        --config <FILE>              Path to a toml file specifying a list of options this will override any other
                                     options set
        --coveralls <KEY>            Coveralls key, either the repo token, or if you're using travis use $TRAVIS_JOB_ID
                                     and specify travis-{ci|pro} in --ciserver
    -e, --exclude <PACKAGE>...       Package id specifications to exclude from coverage. See cargo help pkgid for more
                                     info
        --exclude-files <FILE>...    Exclude given files from coverage results has * wildcard
        --features <FEATURE>...      Features to be included in the target project
        --manifest-path <PATH>       Path to Cargo.toml
    -o, --out <FMT>...               Output format of coverage report [possible values: Json, Toml, Stdout, Xml, Html,
                                     Lcov]
        --output-dir <PATH>          Specify a custom directory to write report files
    -p, --packages <PACKAGE>...      Package id specifications for which package should be build. See cargo help pkgid
                                     for more info
        --report-uri <URI>           URI to send report to, only used if the option --coveralls is used
    -r, --root <DIR>                 Calculates relative paths to root directory. If --manifest-path isn't specified it
                                     will look for a Cargo.toml in root
        --run-types <TYPE>...        Type of the coverage run [possible values: Tests, Doctests, Benchmarks, Examples]
        --target-dir <DIR>           Directory for all generated artifacts
    -t, --timeout <SECONDS>          Integer for the maximum time in seconds without response from test before timeout
                                     (default is 1 minute).

ARGS:
    <args>...    Arguments to be passed to the test executables can be used to filter or skip certain tests
```

## Features

Below is a list of features currently implemented. As Tarpaulin loads binary
files into memory and parses the debugging information, different setups could
lead to coverage not working. In this instance, please raise an issue detailing
your setup and an example project and I'll attempt to fix it (please link us to
a repo and the commit containing your project and paste the verbose output).

* Line coverage
* Uploading coverage to https://coveralls.io or https://codecov.io
* HTML report generation and other coverage report types
* Coverage of tests, doctests, benchmarks and examples possible
* Excluding irrelevant files from coverage
* Config file for mutually exclusive coverage settings (see `Config file` section for details)

## Usage

### Installation

Tarpaulin depends on cargo which depends on SSL. Make sure you've installed
your distros SSL development libraries and they are on your path before
attempting to install tarpaulin. For example for Debian/Ubuntu:

```text
apt-get update && apt-get install libssl-dev pkg-config cmake zlib1g-dev
```

Tarpaulin is a command-line program, you install it into your linux development
environment with cargo install:

```text
cargo install cargo-tarpaulin
```

### Command line

To get detailed help on available arguments when running tarpaulin call:

```text
cargo tarpaulin --help
```
Currently no options are required, if no root directory is defined Tarpaulin
will run in the current working directory.

Below is a Tarpaulin run utilising one of our example projects. This is a
relatively simple project to test and if you check the test, you can see the
output correctly reports the lines the test hits.


```text
cargo tarpaulin -v
[INFO tarpaulin] Running Tarpaulin
[INFO tarpaulin] Building project
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s                                                                                                                                                      
[DEBUG tarpaulin] Processing simple_project
[INFO tarpaulin] Launching test
[INFO tarpaulin] running /home/xd009642/code/rust/tarpaulin/tests/data/simple_project/target/debug/deps/simple_project-b0accf6671d080e0

running 1 test
test tests::bad_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

[INFO tarpaulin] Coverage Results:
|| Uncovered Lines:
|| src/lib.rs: 6
|| src/unused.rs: 4-6
|| Tested/Total Lines:
|| src/lib.rs: 5/6
|| src/unused.rs: 0/3
|| 
55.56% coverage, 5/9 lines covered
```

Tarpaulin can also report the change in coverage for each file between runs. If
the tests were updated in the previous example to cover all the lines we would
expect the following output.

```text
cargo tarpaulin -v
[INFO tarpaulin] Running Tarpaulin
[INFO tarpaulin] Building project
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s                                                                                                                                                      
[DEBUG tarpaulin] Processing simple_project
[INFO tarpaulin] Launching test
[INFO tarpaulin] running /home/xd009642/code/rust/tarpaulin/tests/data/simple_project/target/debug/deps/simple_project-b0accf6671d080e0

running 1 test
test tests::bad_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

[INFO tarpaulin] Coverage Results:
|| Tested/Total Lines:
|| src/lib.rs: 6/6 +16.67%
|| src/unused.rs: 3/3 +100%
|| 
100% coverage, 9/9 lines covered, +44.44% change in coverage
```

Hint: if using coveralls.io with travis-ci run with the options
`--ciserver travis-ci --coveralls $TRAVIS_JOB_ID`. The coveralls.io repo-token
is mainly designed for private repos and it won't generate a badge for the
coverage results submitted (although you can still see them on the coveralls
web interface). For an example of a project using Tarpaulin, you can check out
my crate [keygraph-rs](https://github.com/xd009642/keygraph-rs).

### Ignoring code in files.

Tarpaulin now allows you to ignore modules or functions using config attributes.
Below is an example of ignoring the main function in a project:

```Rust
#[cfg_attr(tarpaulin, skip)]
fn main() {
    println!("I won't be included in results");
}
```

### Continuous Integration Services

Tarpaulin aims to be easy to add to your CI workflow. With well tested support
for Travis-CI it also supports sending CI specific meta-data to coveralls.io for
Circle, Semaphore, Jenkins and Codeship (though only Jenkin's has been tested).

You can also use Tarpaulin on Azure, check out
[crate-ci/azure-pipelines](https://github.com/crate-ci/azure-pipelines) for an
example config.

#### Travis-ci and Coverage Sites

The expected most common usecase is launching coverage via a CI service to
upload to a site like codecov or coveralls. Given the built in support and
ubiquity of travis-ci it seems prudent to document the required steps here for
new users. To follow these steps you'll first need a travis-ci and a project setup
for your coverage reporting site of choice. 

We recommend taking the minimal rust .travis.yml, installing the libssl-dev
dependency tarpaulin has and then running Tarpaulin with the version of 
rustc you require. Tarpaulin is installed in `before_cache` to allow it to be cached
and prevent having to reinstall every Travis run. You can also replace `cargo test`
with a verbose run of tarpaulin to see the test results as well as coverage output.

Tarpaulin is ran after success as there are still some unstable features which could
cause coverage runs to fail. If you don't rely on any of these features you can
alternatively replace `cargo test` with a call to `cargo tarpaulin`.

For codecov.io you'll need to export CODECOV_TOKEN are instructions on this in
the settings of your codecov project.

```yml
language: rust
sudo: required # required for some configurations
# tarpaulin has only been tested on bionic and trusty other distros may have issues
dist: bionic 
addons:
    apt:
        packages:
            - libssl-dev
cache: cargo
rust:
  - stable
  - beta
  - nightly
matrix:
  allow_failures:
    - rust: nightly

before_script: |
  if [[ "$TRAVIS_RUST_VERSION" == stable ]]; then
    cargo install cargo-tarpaulin
  fi

script:
- cargo clean
- cargo build
- cargo test

after_success: |
  if [[ "$TRAVIS_RUST_VERSION" == stable ]]; then
    # Uncomment the following line for coveralls.io
    # cargo tarpaulin --ciserver travis-ci --coveralls $TRAVIS_JOB_ID

    # Uncomment the following two lines create and upload a report for codecov.io
    # cargo tarpaulin --out Xml
    # bash <(curl -s https://codecov.io/bash)
  fi
```

If you rely on certain nightly features you may need to change the `before_script` to
`before_cache` to force tarpaulin to reinstall each time. However, if it can be avoided it
will speed up your CI runs.

Alternatively, there are the prebuilt docker images or the travis-install shell script.
The travis-install script will install the latest tagged release built on travis to your 
travis instance and significantly speeds up the travis builds. You can install via that script 
using `bash <(curl https://raw.githubusercontent.com/xd009642/tarpaulin/master/travis-install.sh)`.

The prebuilt binary is built using github actions ubuntu:latest image, because of this it
doesn't work on xenial or trusty, but it works on bionic. You should still keep the rest
of the recommended travis settings.

### CircleCI

To run tarpaulin on CircleCI you need to run tarpaulin in docker and set the
machine flag to true as shown below:

```yml
jobs:
  coverage:
    machine: true
    steps:
      - checkout
      - run:
          name: Coverage with docker
          command: docker run --security-opt seccomp=unconfined -v "${PWD}:/volume" xd009642/tarpaulin
```

### Gitlab Pipelines

To get the coverage results showing up in your Gitlab pipelines add the following regex to the `Test
coverage parsing` section in the pipelines settings.

```
^\d+.\d+% coverage
```

For installation add `cargo install cargo-tarpaulin -f` to the script section.

### Docker

Tarpaulin has builds deployed to [docker-hub](https://hub.docker.com/r/xd009642/tarpaulin/), 
to run Tarpaulin on any system that has Docker, run this in your project directory:

```text
docker run --security-opt seccomp=unconfined -v "${PWD}:/volume" xd009642/tarpaulin
```

This builds your project inside Docker and runs Tarpaulin without any arguments. There are
also tags available for the latest version on the develop branch in stable or nightly. And 
versions after 0.5.6 will have the latest release built with the rust stable and nightly 
compilers. To get the latest development version built with rustc-nightly run the following:

```text
docker run --security-opt seccomp=unconfined -v "${PWD}:/volume" xd009642/tarpaulin:develop-nightly
```

Note that the build might fail if the Docker image doesn't contain any necessary
dependencies. In that case, you can install dependencies before, like this:

```text
docker run --security-opt seccomp=unconfined -v "${PWD}:/volume" xd009642/tarpaulin sh -c "apt-get install xxx && cargo tarpaulin"
```

### Config file

Tarpaulin has a config file setting where multiple coverage setups can be
encoded in a toml file. This can be provided by an argumnet or if a 
`.tarpaulin.toml` or `tarpaulin.toml` is present in the same directory as
the projects manifest or in the root directory that will be used unless 
`--ignore-config` is passed. Below is an example file:

```toml
[feature_a_coverage]
features = ["feature_a"]

[feature_b_coverage]
features = ["feature_b"]
release = true

[report]
coveralls = "coveralls_key"
out = ["Html", "Xml"]
```

Here we'd create three configurations, one would run your tests with
`feature_a` enabled, and the other with the tests built in release and
`feature_b` enabled. The last configuration uses a reserved configuration name
`report` and this doesn't result in a coverage run but affects the report 
output. This is a reserved feature name and any non-reporting based options
chosen will have no effect on the output of tarpaulin.

For reference on available keys and their types refer to the CLI help text
at the start of the readme or `src/config/mod.rs` for the concrete types
if anything is unclear. For arguments to be passed into the test binary that
follow `--` in tarpaulin use `args` in the toml file.

Setting the field `config` will have no effect on the run as it won't be parsed
for additional configuration.

## Extending Tarpaulin.

There are some tools available which can extend tarpaulin functionality for
other potential user needs.

### Procedural Macros

Normally, Tarpaulin can't report on code coverage within the code for a 
procedural macro. You'll need to add a test that expands the macro at run-time
in order to get those stats. The
[`runtime-macros` crate](https://crates.io/crates/runtime-macros) was made for
this purpose, and its documentation describes how to use it with Tarpaulin.

### Pycobertura 

[`pycobertura`](https://pypi.python.org/pypi/pycobertura) is a python library
for working with cobertura reports. It offers a report diffing tool as well as
it's own report implementations.

To generate a `cobertura.xml` simply run the following tarpaulin command:

```text
cargo tarpaulin --out Xml
```

Then install `pycobertura` with pip and execute the desired command.

As tarpaulin doesn't allow you to change the name of the generated cobertura
report be mindful of this if diffing reports between multiple commits.

## Issues and Contributing

Issues, feature requests and pull requests are always welcome! For a guide on
how to approach bugs found in Tarpaulin and adding features please check 
[CONTRIBUTING](CONTRIBUTING.md).

Rust 1.23 introduced a regression in the compiler affecting tarpaulin's
accuracy. If you see missing lines or files, check your compiler version.

## Roadmap

- [x] Line coverage for tests
- [ ] Branch coverage for tests
- [ ] Condition coverage for tests
- [x] Annotated coverage reports
- [x] Coverage reports in the style of existing tools (i.e. kcov)
- [x] Integration with 3rd party tools like coveralls or codecov
- [x] Optional coverage statistics for doctests (nightly only [tracking issue](https://github.com/rust-lang/rust/issues/56925))
- [ ] MCDC coverage reports
- [ ] OSX support 
- [ ] Windows support

## License

Tarpaulin is currently licensed under the terms of both the MIT license and the
Apache License (Version 2.0). See LICENSE-MIT and LICENSE-APACHE for more details.

## Thanks

I wouldn't have been able to make progress as quickly in this project without
Joseph Kain's blog on writing a debugger in Rust and C. It's a great read, so I
recommend you check it out [here](http://system.joekain.com/debugger/).
