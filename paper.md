---
title: 'Python Dissemination'
tags:
  - Python
  - packaging
  - scripting
  - dependency management
  - conda
authors:
  - name: Rohit Goswami
    orcid: 0000-0002-2393-8056
    affiliation: "1"
affiliations:
 - name: Institute IMX and Lab-COSMO, École polytechnique fédérale de Lausanne (EPFL), Station 12, CH-1015 Lausanne
   index: 1
date: 13 February 2026
bibliography: references.bib
---

## Summary

Python packaging has improved in leaps and bounds over the years. System
dependency management has also become routine through the community-wide
adoption of the `conda` packaging ecosystem. In particular, with the
Rust-based
[cite:&perkelWhyScientistsAre2020](cite:&perkelWhyScientistsAre2020)
stack of `uv` and `pixi`
[cite:&fischerPixiUnifiedSoftware2025](cite:&fischerPixiUnifiedSoftware2025),
it is now possible to rapidly iterate on multi-lingual software with
complex dependencies. "Python Dissemination" constitutes an
intermediate-level curriculum designed to first demonstrate such
scripting, and then the transition from self contained artifacts into
packages to be used by the wider community. Developed within The
Carpentries Workbench infrastructure, the lesson targets scientists who
have mastered basic syntax and are able to interact with libraries [^1]
but lack a rigorous mental model of the software distribution lifecycle.
The curriculum deconstructs the complexities of packaging, separates
abstract dependency declaration from concrete environment resolution,
and can be covered within a single afternoon for dedicated learners,
including an understanding of compiled extensions and their installation
within Python packages.

## Statement of Need

Scripts are the most natural extension of a basic introduction to
programming in Python. Even for those learning on interactive Jupyter
notebooks, the script [^2] forms the unit of communication between
learners. In practice, scripts written in this manner suffer from being
dependent on locally installed environmental dependencies. Modern
tooling does allow for scripts to be self contained even when system
dependencies are involved, and this forms the first part of the lesson.

Beyond this, intermediate courses also rarely address the "packaging
gradient"—the structural transformation required to convert a local
script into a distributable library. Packaging is often left to
tutorials at workshops and other niche setups, often without the benefit
of the pedagogical understanding gained in the last few decades, like
those known from the Carpentries
[cite:&wilsonSoftwareCarpentryGetting2006;&tealDataCarpentryWorkshops2015](cite:&wilsonSoftwareCarpentryGetting2006;&tealDataCarpentryWorkshops2015).

We identify three specific barriers to entry that this curriculum
dismantles:

The Shadowing Fallacy
:   Novices often lack a clear mental model of Python's import
    resolution (`sys.path`), leading to fragile code that breaks when
    moved from the development directory [^3].

The Reproducibility Hysteresis
:   Traditional tooling often conflates **specification** (what the code
    needs) with **resolution** (what the machine installed), leading to
    environments that drift over time.

The Compilation Barrier
:   Compiled extensions dominate performance oriented Python code.
    General-purpose tutorials typically ignore the concepts underlying
    the build systems required to bridge these languages with Python,
    despite the tools themselves having fairly decent documentation.

This lesson adopts a "Good Enough Practices" philosophy
@wilsonGoodEnoughPractices, with tooling for modern standards that
enforce correctness by default.

## Pedagogical Design

The instructional design follows a spiral trajectory, revisiting the
concept of "dependencies" at increasing levels of abstraction, for both
practical scripts to packages meant to be installed by end-users as
shown in Figure ref:fig:pedagogical-gradient. We employ visual mental
models to anchor these concepts.

![The Pedagogical Gradient: Learners progress from self-contained
scripts to structured projects, and finally to distributed
artifacts.](fig/pedagogical_gradient.png){#fig:pedagogical-gradient}

Starting with the most immediate example of a script, we discuss PEP 723
tooling, demonstrating how a script may now carry its own context. By
leveraging system dependencies through `pixi` and Python inline metadata
with `uv` with specific examples, learners gain confidence in the
ability to share reproducible scripts as shown in Figure
ref:fig:hybrid-env.

![The Hybrid Stack: Visualizing the distinction between system binaries
managed by Conda/Pixi and Python libraries managed by
uv.](fig/hybrid_env.png){#fig:hybrid-env}

The curriculum then moves towards a standard self-contained
understanding of the Python import system, using the `src` layout
generated from `uv` and demonstrates how this prevents the "works on my
machine" syndrome caused by implicit relative imports. This is followed
by an understanding of automated testing, linting, formatting, and
release management including challenges for TestPyPI.

A distinct feature of this curriculum involves the explicit handling of
compiled extensions. With the promulgation of Rust-in-Python, along with
the traditional Fortran-in-Python and C/C++-in-Python, many high
performance libraries involve such components. `scikit-build` for CMake
projects, `meson-python` for Meson projects, and the venerable
`setuptools`, all have fairly decent documentation. What is missing,
especially when considering `maturin` and other "all-in-one" project
layouts, is a first principles understanding of the process and logic
behind exposing shared libraries and the use of wheels. We cover this
gap, as demonstrated with an `f2py` example, conceptualized in Figure
ref:fig:build~flow~.

![The Build Pipeline: Transforming compiler-dependent code into
distributable Python artifacts.](fig/build_flow.png)

## Instructional Experience & Infrastructure

We implement this curriculum using The Carpentries Workbench, albeit
with a modified `orgmode` source to generate figures and the `RMarkdown`
required. This infrastructure affords several advantages:

Maintainability
:   The separation of narrative prose from code snippets allows for
    atomic updates to the tooling stack without rewriting the
    pedagogical logic.

Accessibility
:   The workbench rendering ensures the material remains accessible
    across diverse devices, with specialized handling for generating
    all-in-one pages, challenges, hints, key points and other concepts.

The choice of the `uv` and `ruff` toolchain specifically targets the
reduction of "installation friction" in workshops. By utilizing
single-binary tools that require no system-level dependencies, we
minimize the time spent debugging learner environments, maximizing the
time spent on conceptual synthesis. Taken together, we believe this
material empowers learners to generate reproducible results
[cite:&sandveTenSimpleRules2013](cite:&sandveTenSimpleRules2013) and
distribute code effectively as per the FAIR principles
[cite:&barkerIntroducingFAIRPrinciples2022;&communityTuringWayHandbook2019](cite:&barkerIntroducingFAIRPrinciples2022;&communityTuringWayHandbook2019).

[^1]: occasionally assisted in their installation by systems
    administrators

[^2]: sometimes exported from a notebook

[^3]: exacerbated by "inherited" shell configurations
