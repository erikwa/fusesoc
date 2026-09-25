.. _ug_cli:

***************
Running FuseSoC
***************

FuseSoC is a command-line tool; this section explains how to use it.
The following content is aimed at users who already have a hardware design which uses FuseSoC.

Build a design
==============

The ``fusesoc run`` group of commands is used to setup, build, and (if possible) run a design.
The exact actions taken by the individual steps depend on the toolflow.

::

    usage: fusesoc run [-h] [--no-export] [--build-root BUILD_ROOT] [--setup] [--build] [--run] [--target TARGET] [--tool TOOL] [--flag FLAG] [--system-name SYSTEM_NAME] system ...

    positional arguments:
      system                Select a system to operate on
      backendargs           arguments to be sent to backend

    optional arguments:
      -h, --help            show this help message and exit
      --no-export           Reference source files from their current location instead of exporting to a build tree
      --build-root BUILD_ROOT
                            Output directory for build. Defaults to build/$VLNV
      --setup               Execute setup stage
      --build               Execute build stage
      --run                 Execute run stage
      --target TARGET       Override default target
      --tool TOOL           Override default tool for target
      --flag FLAG           Set custom use flags. Can be specified multiple times
      --system-name SYSTEM_NAME
                            Override default VLNV name for system

When FuseSoC is invoked with the run command, it will create an empty working directory called `work_root` internally, where it by default will create all project files, copy all used source files, build and optionally run the project.

Setup, build and run
--------------------

The process of running EDA tools is divided into three steps called *setup*, *build* and *run*. The *setup* stage creates the working directory and all project files. The *build* stage runs one or more EDA tools to build an artifact, e.g. a GDS, simulation model or FPGA image. The *run* stage is only implemented for some tool flows, such as simulation flows where it runs the simulation. Some FPGA flows uses the *run* stage to program an FPGA device.

Normally FuseSoC runs all three stages, but if the `--setup` flag is added, it will stop after the setup stage and if the `--build` flag is set it will stop after the build stage. Many of the newer backends don't need these flags and will instead only run setup or build when input files or options have changed.

Work root
---------
`work_root` is a private working directory and should not be shared between different builds. It is however perfectly fine to reuse the working directory for e.g. running several simulations using different runtime options as long as the build-time options and source files are not modified.

By default, FuseSoC will use `build/<sanitized VLNV>/<target>`, where `<sanitized VLNV>` is the top-level VLNV with underscore instead of colon as the separator. For the Flow API, `<target>` is just the name of the target in the core description file, e.g. `sim`. For the old Tool API, it is a combination of target name and the tool backend, e.g. `sim-verilator`. The work root directory can be changed with the `--work-root` option.

Exporting source files
----------------------

The standard behavior for FuseSoC is to copy all used source files into a subdirectory of the work root. This has three advantages. The work root is self-contained with all the source files and can be copied elsewhere for archival purposes or to build on another machine. No stray files are picked up by mistake from the original source directories. It is always possible to know from exactly which files a build was created. Despite this, there are situations where it is preferable to reference the source files from their original location. This can be done by adding the `--no-export` flag.

.. _ug_cli_core_validate:

Checking core files
===================

The ``fusesoc core validate`` command checks core files for errors without building anything.
It is intended for continuous integration and pre-commit checks of core libraries.

::

    usage: fusesoc core validate [-h] [paths ...]

    positional arguments:
      paths       Core files or directories to check (default: all registered
                  libraries)

Each path can be a core file or a directory, which is searched for core files the same way as a library.
Without any paths, all registered libraries are checked.

Every core file with an error is reported on its own line as ``<core file>: <error>``.
The command exits with a non-zero exit code if any error was found.

The command only reads the core files, which catches errors such as YAML syntax errors, keys or values not allowed by the CAPI schema, a missing or wrong ``CAPI=2:`` first line, and malformed expressions.
Errors that depend on a build, such as dependencies that cannot be resolved, are only found by ``fusesoc run``.

Continuous integration
----------------------

To check all core files in a repository, run the command from the repository root::

    pip install fusesoc
    fusesoc core validate .

pre-commit
----------

FuseSoC provides a `pre-commit <https://pre-commit.com/>`_ hook which checks all changed core files on every commit.
Add it to the ``.pre-commit-config.yaml`` of your repository:

.. code-block:: yaml

    repos:
      - repo: https://github.com/olofk/fusesoc
        rev: <FuseSoC version>
        hooks:
          - id: fusesoc-core-validate

pre-commit installs FuseSoC by itself, so it does not need to be installed beforehand.
Files that end in ``.core`` but are not FuseSoC core files can be skipped with the standard pre-commit ``exclude`` option.
