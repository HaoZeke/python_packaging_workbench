---
title: "Instructor Notes"
---

# Overall Lesson Design

This lesson follows a progressive arc from standalone scripts to
published packages with compiled extensions. Each episode builds on the
previous one, so it is important to teach them in order. The full lesson
spans eight episodes with approximately 3 hours of instruction plus 1.5
hours of exercises, suitable for a full-day workshop. A half-day
workshop can cover the first five episodes (through Release
Engineering), with the remaining three assigned as self-study.

The running example is a small computational chemistry library
(`chemlib`{.verbatim}) that evolves from a single script into a full
package with Fortran extensions. You do not need domain expertise to
teach this lesson; the chemistry content is deliberately minimal and
serves only as a realistic motivating example.

# Before the Workshop

-   Ensure learners have `pixi`{.verbatim} and `uv`{.verbatim} installed
    (see the Setup page).
-   Ask learners to create a [Test
    PyPI](https://test.pypi.org/account/register/) account before
    arriving.
-   Have a working internet connection; several episodes download
    packages.
-   Pre-download the example data files if you expect connectivity
    issues.

# Episode-Specific Notes

## Writing Reproducible Python (20 min teaching / 10 min exercises)

-   Start by asking learners how they currently share scripts. This
    surfaces the pain points that the lesson addresses.
-   The inline script metadata (PEP 723) feature requires
    `uv`{.verbatim} and may be unfamiliar even to experienced Python
    users. Emphasize that this is a new standard and not yet widely
    adopted.
-   The `pixi`{.verbatim} task runner integration is optional but useful
    for showing how system-level dependencies (like `ase`{.verbatim})
    can be pinned alongside Python dependencies.

## Modules, Packages, and The Search Path (15 min teaching / 5 min exercises)

-   Learners often confuse "module" (a single `.py`{.verbatim} file)
    with "package" (a directory with `__init__.py`{.verbatim}). Draw the
    distinction clearly.
-   The `sys.path`{.verbatim} inspection is the key conceptual moment.
    Have learners run it interactively and compare outputs.
-   Common misconception: learners may believe `import`{.verbatim}
    searches the entire filesystem. Clarify that it only searches
    `sys.path`{.verbatim} entries.

## The Project Standard: pyproject.toml (20 min teaching / 10 min exercises)

-   This is the densest episode. Consider a short break before it.
-   The `src`{.verbatim} layout is non-obvious to beginners. Explain
    that it prevents accidental imports from the working directory
    instead of the installed package.
-   Walk through the `uv init`{.verbatim} output slowly; there is a lot
    of generated content.
-   If learners ask about `setup.py`{.verbatim} or
    `setup.cfg`{.verbatim}, acknowledge them as legacy approaches and
    redirect to `pyproject.toml`{.verbatim} as the current standard (PEP
    621).

## Quality Assurance: Testing and Linting (20 min teaching / 10 min exercises)

-   Learners often resist testing because their code "works." Frame
    testing as a way to save time when refactoring or onboarding new
    collaborators.
-   `ruff`{.verbatim} is fast enough to run live. Consider showing a
    side-by-side of before/after formatting.
-   The `--dev`{.verbatim} flag for development dependencies is a
    critical concept. Make sure learners understand why linters and test
    frameworks should not be runtime dependencies.

## Release Engineering: Logs and Artifacts (25 min teaching / 15 min exercises)

-   The `towncrier`{.verbatim} fragment workflow may feel heavyweight
    for small projects. Acknowledge this and explain that it scales well
    for multi-contributor projects.
-   Publishing to Test PyPI is the highlight of this episode. Have
    learners actually publish and then install each other's packages if
    time permits.
-   Common issue: package name collisions on Test PyPI. Have learners
    use unique suffixes (e.g., `chemlib-<username>`{.verbatim}).

## If It Isn't Documented, It Doesn't Exist (25 min teaching / 15 min exercises)

This episode focuses on docstrings and Sphinx. It assumes learners
already have a working, tested package with a release pipeline.

-   Start with docstrings, not Sphinx. The "write a NumPy-style
    docstring" exercise gives learners an immediate, tangible skill even
    if they never set up Sphinx.
-   Sphinx configuration can feel overwhelming. The episode uses a
    minimal `conf.py`{.verbatim} with only three extensions:
    `autoapi`{.verbatim}, `viewcode`{.verbatim}, and
    `intersphinx`{.verbatim}. Resist the urge to add more.
-   The Shibuya theme is recommended for its clean modern look, but any
    theme works. If learners prefer Read the Docs, that is fine.
-   The `intersphinx`{.verbatim} challenge is a "wow" moment for many
    learners. Show how a `` :class:`numpy.ndarray` ``{.verbatim}
    reference in a docstring automatically becomes a hyperlink to the
    NumPy documentation.
-   Common issue: the `sys.path.insert`{.verbatim} line in
    `conf.py`{.verbatim} must point to the correct relative path for
    your `src`{.verbatim} directory. If `autoapi`{.verbatim} finds zero
    modules, this is almost always the cause.
-   The GitHub Pages deployment section introduces GitHub Actions
    workflow files. Keep it brief; the next episode (CI) covers workflow
    files in full detail.
-   The PR preview subsection uses a two-workflow pattern (build +
    comment). If learners ask why, explain the security model: the build
    runs with the PR author's limited permissions, while the comment
    workflow runs with write access only after a successful build. This
    prevents malicious PRs from exploiting elevated permissions.
-   If time is short, the PR preview can be skipped or shown as a demo.
    The core value of the episode is docstrings + Sphinx + autoapi.

## The Gatekeeper: Continuous Integration (20 min teaching / 10 min exercises)

This episode follows naturally from Quality Assurance (local hooks) and
Release Engineering (manual publishing). It closes the loop by
automating both. Learners will have already seen a workflow file in the
Documentation episode, so the YAML structure should not be entirely new.

-   Many learners will not have used GitHub Actions before. Start by
    explaining the YAML structure: `on`{.verbatim} (triggers),
    `jobs`{.verbatim} (what runs), `steps`{.verbatim} (how). The episode
    includes a "Reading the Workflow" challenge for this purpose.
-   The `astral-sh/setup-uv`{.verbatim} action is the recommended way to
    install `uv`{.verbatim} in CI. Emphasize the
    `enable-cache: true`{.verbatim} flag, which dramatically speeds up
    repeated runs.
-   Build up from a single-OS pipeline to the matrix. This avoids
    overwhelming learners with the full YAML in one go.
-   The matrix strategy is the key concept. Show the math: 2 Python
    versions x 3 operating systems = 6 parallel jobs. This is where CI
    earns its keep.
-   The release job at the end ties back to `uvx twine`{.verbatim} from
    Release Engineering. Emphasize that `needs: check`{.verbatim} means
    a broken build is never published.
-   Common issue: Windows path separators. If a learner has hardcoded
    `/`{.verbatim} in file paths, the Windows job will fail. This loops
    back to `pathlib`{.verbatim} from the very first episode.
-   If time is short, skip the matrix and just show the basic single-OS
    pipeline. The matrix can be assigned as homework.

## Compiled Extensions: The Shared Object Pipeline (35 min teaching / 15 min exercises)

This is the most advanced episode and serves as the capstone of the
lesson. By this point learners have the full toolchain (packaging,
testing, docs, CI) and are ready to tackle the binary distribution
problem.

-   Gauge the audience and consider making it optional for a shorter
    workshop.
-   Learners without Fortran experience should not worry; the Fortran
    code is provided and the focus is on the build pipeline, not the
    language.
-   The `meson-python`{.verbatim} backend is less common than
    `setuptools`{.verbatim}. Explain that it exists specifically for
    compiled code and is used by `numpy`{.verbatim} and
    `scipy`{.verbatim}.
-   `cibuildwheel`{.verbatim} can take a long time to run in CI.
    Consider showing pre-recorded output if live demos are not feasible.
    Learners now understand CI from the previous episode, so the
    `cibuildwheel`{.verbatim} matrix diagram should feel familiar.
-   The "manual install" challenge (copying `.so`{.verbatim} into
    `site-packages`{.verbatim}) is deliberately low-level. It
    demystifies what `pip install`{.verbatim} actually does under the
    hood and helps learners appreciate why build tools exist.
-   If a learner's system lacks a Fortran compiler, they can still
    follow along conceptually; the `f2py`{.verbatim} output is shown in
    the lesson.

# Suggested Workshop Schedules

## Half-Day Workshop (3.5 hours, episodes 1-5)

  Time            Activity
  --------------- ---------------------------------
  09:00 - 09:30   Reproducible Python (Scripts)
  09:30 - 09:50   Modules and the Search Path
  09:50 - 10:00   **Break**
  10:00 - 10:30   pyproject.toml with uv
  10:30 - 11:00   Quality Assurance
  11:00 - 11:10   **Break**
  11:10 - 11:50   Release Engineering
  11:50 - 12:30   Q&A / Apply to your own project

This covers the core "script to published package" arc. Documentation,
CI, and Compiled Extensions can be assigned as self-study or covered in
a follow-up session.

## Shorter Session (2 hours, episodes 1-4)

Cover Scripts through Quality Assurance. This gives learners a complete
workflow from script to tested package and can be done without internet
access (no publishing step).

## Full-Day Workshop (all 8 episodes)

  Time            Activity
  --------------- ---------------------------------
  09:00 - 09:30   Reproducible Python (Scripts)
  09:30 - 09:50   Modules and the Search Path
  09:50 - 10:00   **Break**
  10:00 - 10:30   pyproject.toml with uv
  10:30 - 11:00   Quality Assurance
  11:00 - 11:10   **Break**
  11:10 - 11:50   Release Engineering
  11:50 - 12:30   Documentation
  12:30 - 13:30   **Lunch**
  13:30 - 14:00   Continuous Integration
  14:00 - 14:50   Compiled Extensions
  14:50 - 15:00   **Break**
  15:00 - 15:30   Apply to your own project (Q&A)

# Timing Guide

  Episode                  Teaching      Exercises    Total
  ------------------------ ------------- ------------ -------------
  Reproducible Python      20 min        10 min       30 min
  Modules                  15 min        5 min        20 min
  pyproject.toml & uv      20 min        10 min       30 min
  Quality Assurance        20 min        10 min       30 min
  Release Engineering      25 min        15 min       40 min
  Documentation            25 min        15 min       40 min
  Continuous Integration   20 min        10 min       30 min
  Compiled Extensions      35 min        15 min       50 min
  **Total**                **180 min**   **90 min**   **270 min**

For a half-day workshop (3.5 hours with breaks), the first five episodes
(through Release Engineering) fit comfortably. For a shorter session,
stop after Quality Assurance. For a full-day workshop, cover all eight
episodes with Documentation and CI in the late morning and Compiled
Extensions after lunch.

# Common Pitfalls

-   Learners on Windows may have PATH issues with `pixi`{.verbatim} and
    `uv`{.verbatim} after installation. Have them restart their
    terminal.
-   Virtual environment confusion: some learners may have
    `conda`{.verbatim} environments active that interfere with
    `uv`{.verbatim}. Ask them to deactivate first.
-   The `editable install`{.verbatim} concept
    (`uv pip install -e .`{.verbatim}) is powerful but can confuse
    learners who expect changes to require reinstallation. Clarify when
    editable installs do and do not pick up changes (new files require
    reinstall).
-   TestPyPI name collisions: package names are global. Have learners
    append a unique suffix (e.g., `chemlib-<username>`{.verbatim}) to
    avoid conflicts.
-   Fortran compiler availability: not all systems have
    `gfortran`{.verbatim} installed. On macOS, learners may need to
    install it via `brew install gcc`{.verbatim} or use
    `pixi`{.verbatim} to provide it. On Windows, `pixi`{.verbatim} is
    the recommended path.
-   The `uv run`{.verbatim} vs `python`{.verbatim} distinction trips up
    learners who are used to activating virtual environments. Emphasize
    that `uv run`{.verbatim} is the "batteries included" way to execute
    code in the project's environment.
-   If GitHub Actions workflows do not trigger, check that the YAML file
    is in the default branch and that the file is valid YAML
    (indentation matters).
-   `ruff`{.verbatim} version pinning: the
    `.pre-commit-config.yaml`{.verbatim} pins a `ruff`{.verbatim}
    version. If learners see different lint results locally vs in CI,
    version mismatch is usually the cause.

# Accessibility and Inclusivity

-   All diagrams in the lesson have alt-text descriptions. If projecting
    slides, read the diagram aloud for visually impaired learners.
-   The lesson uses colour-coded diagrams. Ensure your projector has
    adequate contrast, and point out the labels rather than relying on
    colour alone.
-   Terminal font size: increase your terminal font to at least 18pt
    when live-coding so learners at the back can follow along.
-   Provide the example data files on a USB stick or shared drive if
    connectivity is unreliable.
