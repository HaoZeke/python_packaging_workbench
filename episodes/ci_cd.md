---
title: "The Gatekeeper: Continuous Integration"
teaching: 20
exercises: 10
---

::: questions
-   What happens if I forget to run the tests before pushing?
-   How do I ensure my code works on Windows, Linux, and macOS?
-   How do I automate `uv`{.verbatim} in the cloud?
:::

::: objectives
-   Create a GitHub Actions workflow file that lints and tests on every
    push.
-   Configure the `astral-sh/setup-uv`{.verbatim} action for cached,
    high-performance CI.
-   Define a test matrix to validate code across multiple Python
    versions and operating systems.
-   Connect CI to the release pipeline from the Release Engineering
    episode.
:::

# The Limits of Local Hooks

In the Quality Assurance episode, we installed `prek`{.verbatim} to run
`ruff`{.verbatim} before every commit. That is a good first line of
defence, but it has gaps:

-   A collaborator can bypass hooks with
    `git commit --no-verify`{.verbatim}.
-   Hooks only run on **your** machine, with **your** operating system
    and Python version.
-   If it works on your MacBook but breaks on a colleague's Linux
    cluster, you will not find out until they complain.

**Continuous Integration (CI)** closes these gaps by running your test
suite on a neutral server every time code is pushed. It is the
"gatekeeper" that protects the `main`{.verbatim} branch.

```{=org}
#+RESULTS[101b528c6cf264381dd12c73180ae27bad4d61de]:
```
![Flowchart showing a developer pushing code, GitHub Actions running checkout, install, lint, and test steps, then either allowing merge or blocking](fig/ci-gatekeeper-flow.png)

# Anatomy of a Workflow File

GitHub Actions reads YAML files from `.github/workflows/`{.verbatim}.
Each file describes **when** to run (`on`{.verbatim}), **what machine**
to use (`runs-on`{.verbatim}), and **what commands** to execute
(`steps`{.verbatim}).

Let's create our gatekeeper. Start by making the directory:

``` bash
mkdir -p .github/workflows
```

Now create `.github/workflows/ci.yml`{.verbatim} with the following
content. This mirrors exactly what we did locally in the Quality
Assurance episode: lint, then test.

    name: CI

    on:
      push:
        branches: [main]
      pull_request:

    jobs:
      check:
        name: Lint and Test
        runs-on: ubuntu-latest
        steps:
          - name: Checkout Code
            uses: actions/checkout@v4

          - name: Install uv
            uses: astral-sh/setup-uv@v5
            with:
              enable-cache: true

          - name: Set up Python
            run: uv python install 3.12

          - name: Install Dependencies
            run: uv sync --all-extras --dev

          - name: Lint
            run: uv run ruff check .

          - name: Format Check
            run: uv run ruff format --check .

          - name: Test
            run: uv run pytest --cov=src

::: callout
The `astral-sh/setup-uv`{.verbatim} action installs `uv`{.verbatim} and
(with `enable-cache: true`{.verbatim}) caches the downloaded packages
between runs. This makes subsequent CI runs significantly faster than a
fresh install each time.
:::

::: challenge
## Challenge: Reading the Workflow

Before we push anything, make sure you understand the structure. Answer
the following:

1.  Which event triggers this workflow on a pull request?
2.  What operating system does the job run on?
3.  Why do we use `ruff format --check`{.verbatim} instead of
    `ruff format`{.verbatim}?

::: solution
1.  The `pull_request`{.verbatim} trigger (under `on:`{.verbatim}) fires
    whenever a PR is opened or updated against any branch.
2.  `ubuntu-latest`{.verbatim} (a Linux virtual machine hosted by
    GitHub).
3.  `--check`{.verbatim} exits with an error if files **would** be
    reformatted, without actually modifying them. In CI we want to
    **detect** problems, not silently fix them. The developer should run
    `ruff format`{.verbatim} locally and commit the result.
:::
:::

# The Test Matrix

The workflow above runs on one OS with one Python version. That is
better than nothing, but one of the biggest risks in scientific Python
is **compatibility**.

-   A script might work on Linux but fail on Windows due to path
    separators (`/`{.verbatim} vs `\`{.verbatim}).
-   Code might work on Python 3.12 but fail on 3.11 because it uses a
    feature added in 3.12 (like `type`{.verbatim} statement syntax).
-   A filename like `aux.py`{.verbatim} is perfectly legal on Linux but
    reserved on Windows.

A **Matrix Strategy** tells GitHub to run the same job across every
combination of parameters. We define the axes (Python versions,
operating systems) and GitHub spins up one runner per combination.

Replace the `jobs:`{.verbatim} block in your `ci.yml`{.verbatim} with
the version below. The `steps`{.verbatim} remain identical; only the job
header changes.

    name: CI

    on:
      push:
        branches: [main]
      pull_request:

    jobs:
      check:
        name: Test on ${{ matrix.os }} / Py ${{ matrix.python-version }}
        runs-on: ${{ matrix.os }}
        strategy:
          fail-fast: false
          matrix:
            python-version: ["3.11", "3.12"]
            os: [ubuntu-latest, windows-latest, macos-latest]

        steps:
          - uses: actions/checkout@v4

          - uses: astral-sh/setup-uv@v5
            with:
              enable-cache: true

          - name: Install Python ${{ matrix.python-version }}
            run: uv python install ${{ matrix.python-version }}

          - name: Install Dependencies
            run: uv sync --all-extras --dev

          - name: Lint
            run: uv run ruff check .

          - name: Format Check
            run: uv run ruff format --check .

          - name: Test
            run: uv run pytest --cov=src

Two Python versions times three operating systems gives **six parallel
jobs**. If any single job fails, the Pull Request is blocked.

```{=org}
#+RESULTS[70f95fccaa944451588cb37e5a915ed886cbbb6c]:
```
![](../../episodes/fig/ci-matrix.png)

![Diagram showing a git push triggering six parallel CI jobs across two Python versions and three operating systems, all feeding into a merge decision](fig/ci-matrix.png)

::: challenge
## Challenge: The Windows Path Bug

Consider the following line in `chemlib`{.verbatim}:

``` python
data_path = "src/chemlib/data/file.txt"
```

1.  Why would this fail on the `windows-latest`{.verbatim} runner?
2.  Rewrite it using `pathlib`{.verbatim} so it works on all three
    operating systems.
3.  Which episode's key lesson does this reinforce?

::: solution
1.  Windows uses backslash (`\`{.verbatim}) as the path separator. A
    hardcoded forward slash string will not resolve correctly on
    Windows.

2.  Use `pathlib.Path`{.verbatim}:

    ``` python
    from pathlib import Path
    data_path = Path("src") / "chemlib" / "data" / "file.txt"
    ```

3.  The very first episode (Writing Reproducible Python), where we
    introduced `pathlib`{.verbatim} for cross-platform file handling.
:::
:::

# Connecting CI to Releases

In the Release Engineering episode, we manually uploaded artifacts to
TestPyPI with `uvx twine`{.verbatim}. We also previewed an automated
release job. Now that we understand how workflows are structured, let's
see the complete picture.

Add a second job to the same `ci.yml`{.verbatim} file. This job only
runs when you push a version tag (e.g., `v0.1.0`{.verbatim}) and only
**after** the test matrix passes.

``` yaml
release:
  needs: check
  if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v')
  runs-on: ubuntu-latest
  permissions:
    id-token: write
    contents: read

  steps:
    - uses: actions/checkout@v4
    - uses: astral-sh/setup-uv@v5

    - name: Build
      run: uv build

    - name: Publish to TestPyPI
      uses: pypa/gh-action-pypi-publish@release/v1
      with:
        repository-url: https://test.pypi.org/legacy/
```

Key details:

`needs: check`{.verbatim}
:   The release job waits for all six matrix jobs to pass. A broken
    build is never published.

`id-token: write`{.verbatim}
:   Enables **OIDC Trusted Publishing**. GitHub proves its identity to
    PyPI directly, so you never need to store an API token as a secret.

The tag filter
:   Only tags starting with `v`{.verbatim} (like `v0.1.0`{.verbatim})
    trigger the release. Normal pushes to `main`{.verbatim} run tests
    but do not publish.

::: challenge
## Challenge: The Release Workflow

Walk through the following scenario:

1.  You merge a pull request to `main`{.verbatim}. Does the
    `release`{.verbatim} job run?
2.  You tag the merge commit with `git tag v0.2.0`{.verbatim} and
    `git push --tags`{.verbatim}. What happens now?
3.  Imagine the `windows-latest / Py 3.11`{.verbatim} job fails. Does
    the release still happen?

::: solution
1.  **No.** The `if:`{.verbatim} condition requires the ref to start
    with `refs/tags/v`{.verbatim}. A push to `main`{.verbatim} does not
    match.
2.  The tag push triggers CI. All six matrix jobs run. If they pass, the
    `release`{.verbatim} job runs: it builds the wheel and sdist, then
    publishes to TestPyPI via OIDC.
3.  **No.** The `needs: check`{.verbatim} dependency means the
    `release`{.verbatim} job is skipped when any matrix job fails. The
    tag remains, and you can re-trigger after fixing the issue.
:::
:::

::: keypoints
-   **Continuous Integration** runs your test suite on a neutral server
    on every push, catching problems that local hooks miss.
-   `astral-sh/setup-uv`{.verbatim} provides a cached, high-performance
    `uv`{.verbatim} environment in GitHub Actions.
-   A **Matrix Strategy** tests across multiple operating systems and
    Python versions in parallel.
-   CI can gate releases: the `release`{.verbatim} job uses
    `needs:`{.verbatim} to ensure tests pass before publishing.
:::
