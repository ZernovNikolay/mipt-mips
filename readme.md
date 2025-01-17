[![Build Status](https://travis-ci.com/MIPT-ILab/mipt-mips.svg?branch=master)](https://travis-ci.com/MIPT-ILab/mipt-mips)
[![Build status](https://ci.appveyor.com/api/projects/status/3a8h619rhn3pcdlm?svg=true)](https://ci.appveyor.com/project/pavelkryukov/mipt-mips)
[![codecov](https://codecov.io/gh/MIPT-ILab/mipt-mips/branch/master/graph/badge.svg)](https://codecov.io/gh/MIPT-ILab/mipt-mips)
[![CodeFactor](https://www.codefactor.io/repository/github/mipt-ilab/mipt-mips/badge)](https://www.codefactor.io/repository/github/mipt-ilab/mipt-mips)
[![Github Releases](https://img.shields.io/github/release/MIPT-ILab/mipt-mips.svg)](https://github.com/MIPT-ILab/mipt-mips/releases)

# MIPT-MIPS / MIPT-V

MIPT-MIPS / MIPT-V is a pre-silicon simulator of MIPS and RISC-V CPU. It measures _performance_ of program running on CPU, thus taking best features of RTL and common functional simulation:
* **Precision**. We provide cycle-accurate models of branch prediction unit, pipeline, and other hardware internals.
* **Customization**. Cache size, branch prediction algorithms, and other parameters can be easily changed even to unfeasible modes.
* **Simplicity**. Our source files are much more readable than RTL and independent on SDK and synthesis flow.
* **Speed**. Megaherz simulation frequency on i5-7300U.
* **Scalability**. Modularized structure allows integration of more microarchitecture configurations.

MIPT-MIPS / MIPT-V can be used for different purposes:
* Performance control of **software optimizations**: you may check what and why happened to IPC.
* Pathfinding of **hardware optimizations**: you may easily integrate some nice feature to CPU model.
* **Comparison** of hardware solutions from different vendors.
* **Performance control** of developed or produced hardware.
* **Education**: simulator is a nice experimental frog to study CPU internals and software development process.

Key system-level features:
* Compatibility with interactive [MARS](http://courses.missouristate.edu/KenVollmar/mars/) system calls.
* Interactive simulation with [GDB](https://github.com/MIPT-ILab/mipt-mips/wiki/Interactive-simulation-with-GDB)

Key microarchitecture features:
* Configurable [branch prediction unit](https://github.com/MIPT-ILab/mipt-mips/wiki/BPU-model) with several prediction algorithms
* Configurable instruction cache
* Interstage data bypassing

More details about internals are available on [Wiki](https://github.com/MIPT-ILab/mipt-mips/wiki/Home/)

## Requirements

We use C++17 features and Boost 1.61. Thus, you have to [use compilers](https://github.com/MIPT-ILab/mipt-mips/wiki/C-compilers) of these versions or newer:
* GCC 7
* Clang 5.0
* Apple LLVM Version 10.0.0
* MS Visual Studio 2017 (Boost 1.66 and CMake 3.10.2 are required)

[Install Boost](https://github.com/MIPT-ILab/mipt-mips/wiki/Required-libraries) before building the project.

To work with MIPS traces, you need to install MIPS binutils. Please follow [our manual](https://github.com/MIPT-ILab/mipt-mips/wiki/MIPS-binutils) if you are using Linux, OS X, or Windows.

To work with RISC-V traces, you need to install RISC-V toolchain. Please follow [the official instruction](https://github.com/riscv/riscv-gnu-toolchain/blob/master/README.md).

Our build system is CMake. You should install CMake 3.9 or higher.
Check [our Wiki page](https://github.com/MIPT-ILab/mipt-mips/wiki/CMake) to get more details about CMake.
Users of IDE (Visual Studio, Eclipse, CodeBlocks etc.) may generate project files with CMake as well.

To generate RISC-V opcodes, CMake uses Python. Python interpreter should be available in your environment.
If you still use Python 2, be sure you have `future` package installed: `pip install --user future`.

## Command line options

### Standalone run options

* `-b <filename>` — provide path to ELF binary file to execute.
* `-n <number>` — number of instructions to run. If omitted, simulation continues until halting system call or jump to `null` is executed.

### ISA and system-level options:

* `-I` — modeled ISA. Default version is `mars`.
    * `mips32`, `mips64` — state-of-the-art MIPS
    * `riscv32`, `riscv64`, `riscv128` — RISC-V with all instructions
    * `spim`, `spim64` — simplified MIPS without delayed branches
    * `mipsI`, `mipsII`, `mipsIII`, `mipsIV` — legacy MIPS versions
* `-f` — enables functional simulation only
* `--mars` — enables MARS-compatible mode of system calls

### Outputs
* `-l` — enables per-module output, for instance:
    * `-l fetch,decode` — prints only fetch and decode stages
    * `-l cpu` —  prints all stages
    * `-l cpu,!mem` —  print all except mem stage
* `-d` — enables output of functional simulator
* `--tdump` — enables module topology dump into topology.json

### Performance mode options

#### Branch prediction
* `--bp-mode` — prediction mode. Check supported modes in [manual](https://github.com/MIPT-ILab/mipt-mips/wiki/BPU-model)
* `--bp-lru` — prediction replacement policy: _LRU_, _pseudo-LRU_, or _infinite_
* `--bp-size` — branch prediction cache size (amount of tracked branch instructions)
* `--bp-ways` — # of ways in branch prediction cache

#### Instruction cache
* `--icache-type` — instruction cache type: _LRU_, _pseudo-LRU_, _always-hit_, or _infinite_
* `--icache-size` — instruction cache size in bytes
* `--icache-ways` — # of ways in instruction cache
* `--icache-line-size` — line size of instruction cache

#### Execution pipeline
* `--long-alu-latency` - number of execution stages required for long arithmetic instructions to be complete

## Workflow example

#### Clone
  1. Check that your environment meets all the requirements above.
  1. Clone repository with submodules: `git clone --recursive https://github.com/MIPT-ILab/mipt-mips.git`
#### Build
To build simulator faster, we recommend to [install Ninja](https://github.com/ninja-build/ninja/wiki/Pre-built-Ninja-packages).
  1. Create a **new** build directory somewhere, then cd into it: `mkdir /path/to/your/build/directory`
  1. Go to the build directory: `cd /path/to/your/build/directory`
  1. Run `cmake /path/to/mipt-mips/simulator -G "Ninja"` to configure CMake
  1. Run `ninja` to get the `mipt-mips` binary file
  1. If you changed some source code files, just type `ninja` to rebuild project
#### Run
  1. Now you can run simulation: `./mipt-mips -b /path/to/binary`
  1. See more command line options in the paragraph below
  1. To run all unit tests, call `ninja unit-tests && ctest --verbose -C Release` from your build directory.

## About MIPT-MIPS / MIPT-V

[![](https://github.com/MIPT-ILab/ca-lectures/blob/master/images/mipt-eng.jpg?raw=true)](https://mipt.ru/english)
[![](https://github.com/MIPT-ILab/ca-lectures/blob/master/images/drec.gif?raw=true)](https://mipt.ru/drec/)
[![Logo](https://avatars2.githubusercontent.com/u/13999586?s=100)](https://mipt-ilab.github.io/)

This project is a part of [ILab](https://mipt-ilab.github.io/) activity at [Moscow Institute of Physics and Technology](http://phystech.edu/) (MIPT).

The main goal of the project is to teach the students the computer architecture through development of a microprocessor implementing the [MIPS](http://en.wikipedia.org/wiki/MIPS32) and [RISC-V](http://en.wikipedia.org/wiki/RISC-V)  instruction set in both functional and performance simulators.

### May I contribute?

Yes, if you attend lectures on Computer Architecture. See our [contributing.md](contributing.md) file for details.
