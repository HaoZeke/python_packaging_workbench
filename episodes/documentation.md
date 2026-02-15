---
title: "If It Isn't Documented, It Doesn't Exist"
teaching: 25
exercises: 15
---

::: questions
-   How do I generate professional documentation for my Python package?
-   What are docstrings, and how do they become web pages?
-   How do I link my documentation to other projects like NumPy?
:::

::: objectives
-   Write NumPy-style docstrings for functions and modules.
-   Configure `Sphinx`{.verbatim} with `autoapi`{.verbatim} to generate
    an API reference automatically.
-   Build HTML documentation locally and verify it.
-   Deploy documentation to the web using GitHub Pages.
:::

# The Documentation Gap

Our `chemlib`{.verbatim} package now has tests, a changelog, and a
release pipeline. A collaborator can install it from TestPyPI with a
single command. But when they type `import chemlib`{.verbatim}, how do
they know what functions are available? What arguments does
`center_of_mass`{.verbatim} expect? What does it return?

Without documentation, the only option is to read the source code. That
works for you (the author), but it does not scale. New users, reviewers,
and your future self all benefit from a browsable, searchable reference.

```{=org}
#+RESULTS[05f9cf2ac8b7aebc44cfaff010715962487ba99e]:
```
![Diagram showing source code with docstrings and conf.py feeding into Sphinx with autoapi, producing an HTML site with index and API reference pages](fig/docs-pipeline.png)

# Writing Good Docstrings

The foundation of automated documentation is the **docstring**: a string
literal that appears as the first statement of a function, class, or
module. Python stores it in the `__doc__`{.verbatim} attribute, and
tools like Sphinx extract it to build reference pages.

The NumPy docstring style is the most common in scientific Python. Let's
update `src/chemlib/geometry.py`{.verbatim} with proper docstrings:

    import numpy as np


    def center_of_mass(atoms):
        """Calculate the geometric center of mass of a set of atoms.

        Parameters
        ----------
        atoms : list of list of float
            A list of 3D coordinates, where each element is ``[x, y, z]``.

        Returns
        -------
        numpy.ndarray
            The mean position as a 1D array of shape ``(3,)``.

        Examples
        --------
        >>> center_of_mass([[0, 0, 0], [2, 0, 0]])
        array([1., 0., 0.])
        """
        data = np.array(atoms)
        return np.mean(data, axis=0)

::: callout
You can verify docstrings interactively at any time:

``` bash
uv run python -c "from chemlib.geometry import center_of_mass; help(center_of_mass)"
```
:::

::: challenge
## Challenge: Document a Second Function

Imagine you add a helper function `distance`{.verbatim} to
`chemlib/geometry.py`{.verbatim} that computes the Euclidean distance
between two points. Write a complete NumPy-style docstring for it.

Your docstring should include:

1.  A one-line summary.
2.  A `Parameters`{.verbatim} section with types.
3.  A `Returns`{.verbatim} section.
4.  An `Examples`{.verbatim} section.

::: solution
``` python
def distance(r_a, r_b):
    """Compute the Euclidean distance between two points.

    Parameters
    ----------
    r_a : array_like
        Coordinates of the first point, shape ``(n,)``.
    r_b : array_like
        Coordinates of the second point, shape ``(n,)``.

    Returns
    -------
    float
        The Euclidean distance between ``r_a`` and ``r_b``.

    Examples
    --------
    >>> distance([0.0, 0.0, 0.0], [1.0, 1.0, 1.0])
    1.7320508075688772
    """
    data_a = np.array(r_a)
    data_b = np.array(r_b)
    return float(np.linalg.norm(data_a - data_b))
```
:::
:::

# Setting Up Sphinx

**Sphinx** is the standard documentation generator in the Python
ecosystem. It reads source files (reStructuredText or Markdown), follows
imports into your package, and produces HTML, PDF, or ePub output.

We will use a **dependency group** (introduced in the
`pyproject.toml`{.verbatim} episode) to keep documentation tools
separate from runtime and dev dependencies:

``` bash
uv add --group docs sphinx sphinx-autoapi shibuya
```

This creates a `docs`{.verbatim} group in `pyproject.toml`{.verbatim}:

``` toml
[dependency-groups]
dev = ["ruff>=0.1.0", "pytest>=8.0.0"]
docs = ["sphinx>=8.0", "sphinx-autoapi>=3.0", "shibuya>=2024.0"]
```

Now initialise the documentation skeleton:

``` bash
mkdir -p docs/source
```

Create the Sphinx configuration file `docs/source/conf.py`{.verbatim}:

    import os
    import sys

    # -- Path setup --------------------------------------------------------------
    sys.path.insert(0, os.path.abspath("../../src"))

    # -- Project information -----------------------------------------------------
    project = "chemlib"
    author = "Your Name"

    # -- Extensions --------------------------------------------------------------
    extensions = [
        "autoapi.extension",       # Auto-generates API pages from source
        "sphinx.ext.viewcode",     # Adds [source] links to API docs
        "sphinx.ext.intersphinx",  # Cross-links to NumPy, Python docs
    ]

    # -- AutoAPI -----------------------------------------------------------------
    autoapi_dirs = ["../../src"]   # Where to find the package source
    autoapi_type = "python"

    # -- Intersphinx -------------------------------------------------------------
    intersphinx_mapping = {
        "python": ("https://docs.python.org/3", None),
        "numpy": ("https://numpy.org/doc/stable", None),
    }

    # -- Theme -------------------------------------------------------------------
    html_theme = "shibuya"

## What Each Extension Does

`autoapi`{.verbatim}
:   Scans your `src/`{.verbatim} directory and builds API reference
    pages for every module, class, and function. You do not need to
    write `.rst`{.verbatim} files by hand.

`viewcode`{.verbatim}
:   Adds "\[source\]" links next to each documented object, letting
    readers jump to the implementation.

`intersphinx`{.verbatim}
:   Enables cross-project linking. When you write
    `` :class:\`numpy.ndarray\` ``{.verbatim} in a docstring, Sphinx
    automatically links to the NumPy documentation.

Finally, create a minimal `docs/source/index.rst`{.verbatim}:

    chemlib
    =======

    A small computational chemistry library used to learn Python packaging.

    .. toctree::
       :maxdepth: 2

       autoapi/index

# Building the Documentation

With everything in place, build the HTML output:

``` bash
uv run --group docs sphinx-build -b html docs/source docs/build/html
```

``` example
...
[autoapi] Reading files with sphinx-autoapi
[autoapi] Found package: chemlib
...
build succeeded.

The HTML pages are in docs/build/html.
```

Open `docs/build/html/index.html`{.verbatim} in your browser. You should
see a clean site with an "API Reference" section listing every function
and its docstring.

::: challenge
## Challenge: The Missing Docstring

1.  Open the generated API reference page in your browser.
2.  Find a function that has **no** docstring (or only a placeholder).
3.  Add a proper NumPy-style docstring to it in the source code.
4.  Rebuild with the `sphinx-build`{.verbatim} command above.
5.  Refresh the page and verify the docstring appears.

::: solution
The rebuild cycle is:

1.  Edit `src/chemlib/geometry.py`{.verbatim} (or whichever module).

2.  Run:

    ``` bash
    uv run --group docs sphinx-build -b html docs/source docs/build/html
    ```

3.  Refresh the browser.

Because `autoapi`{.verbatim} re-reads the source on every build, changes
appear immediately without reinstalling the package.
:::
:::

# Deploying to the Web

Building locally is useful during development, but you ultimately want
the documentation available online. The most common approach in the
Python ecosystem is **GitHub Pages**, deployed automatically via GitHub
Actions.

The next episode covers Continuous Integration in detail. For now, the
key idea is that GitHub Actions can run commands automatically when you
push code. The pattern for documentation deployment is:

1.  **Build:** Check out the code, install the `docs`{.verbatim} group,
    run `sphinx-build`{.verbatim}.
2.  **Upload:** Save the built HTML as a workflow artifact.
3.  **Deploy:** On pushes to `main`{.verbatim}, publish the artifact to
    GitHub Pages.

```{=html}
<!-- -->
```
    name: Documentation

    concurrency:
      group: "pages"
      cancel-in-progress: true

    on:
      push:
        branches: [main]
      pull_request:

    jobs:
      docs:
        runs-on: ubuntu-latest
        permissions:
          contents: write

        steps:
          - uses: actions/checkout@v4
            with:
              fetch-depth: 0

          - uses: astral-sh/setup-uv@v5

          - name: Install dependencies
            run: uv sync --group docs

          - name: Build documentation
            run: uv run sphinx-build -b html docs/source docs/build/html

          - name: Upload artifact
            uses: actions/upload-artifact@v4
            with:
              name: documentation
              path: docs/build/html

          - name: Deploy to GitHub Pages
            if: github.event_name == 'push' && github.ref == 'refs/heads/main'
            uses: peaceiris/actions-gh-pages@v4
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              publish_dir: docs/build/html

::: callout
On pull requests the workflow builds the documentation and reports
success or failure, but does **not** deploy. This ensures broken docs
never reach the live site, while still catching build errors before
merge.
:::

## PR Previews

Catching a broken build is useful, but what about **visual**
regressions—a mangled table, a missing image, or a heading that renders
wrong? You cannot tell from a green checkmark alone.

A **PR preview** solves this: a second workflow waits for the docs build
to finish, then posts a comment on the Pull Request with a link to a
temporary hosted copy of the HTML output. Reviewers can click the link
and see exactly what the documentation will look like before merging.

```{=org}
#+RESULTS[74d350f229fe7f6ee5fe37ad25a660d8fdf68879]:
```
![Diagram showing a PR triggering docs.yml, which uploads an artifact, then pr_comment.yml downloading it and posting a preview link as a PR comment](fig/pr-preview-flow.png)

Add this second workflow file
`.github/workflows/pr_comment.yml`{.verbatim}:

    name: Comment on pull request

    on:
      workflow_run:
        workflows: ["Documentation"]
        types: [completed]

    jobs:
      pr_comment:
        if: >-
          github.event.workflow_run.event == 'pull_request' &&
          github.event.workflow_run.conclusion == 'success'
        runs-on: ubuntu-latest
        permissions:
          pull-requests: write
          issues: write
          actions: read

        steps:
          - uses: HaoZeke/doc-previewer@v0.0.1
            with:
              workflow_run_id: ${{ github.event.workflow_run.id }}
              head_sha: ${{ github.event.workflow_run.head_sha }}
              artifact_name: documentation

This uses a `workflow_run`{.verbatim} trigger, which means it only fires
**after** `docs.yml`{.verbatim} completes. The two-workflow pattern is a
deliberate security measure: the build workflow runs with the PR
author's (limited) permissions, while the comment workflow runs with
write access to post on the PR. This prevents a malicious PR from using
elevated permissions during the build step.

::: challenge
## Challenge: Intersphinx in Action

Add the following docstring to a function in `chemlib`{.verbatim}:

``` python
"""
Returns
-------
numpy.ndarray
    The result as a :class:`numpy.ndarray`.
"""
```

Rebuild the docs and inspect the output.

1.  Does the word `numpy.ndarray`{.verbatim} become a hyperlink?
2.  Where does the link point to?

::: solution
1.  **Yes.** Sphinx resolves the
    `` :class:\`numpy.ndarray\` ``{.verbatim} role using the intersphinx
    inventory downloaded from `numpy.org`{.verbatim}.
2.  The link points to
    `https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html`{.verbatim}.

This works because we configured `intersphinx_mapping`{.verbatim} with
the NumPy docs URL in `conf.py`{.verbatim}. Sphinx fetches a small
inventory file (`objects.inv`{.verbatim}) from that URL at build time
and uses it to resolve cross-references.
:::
:::

::: keypoints
-   **Docstrings** are the raw material for documentation. Use the NumPy
    style (`Parameters`{.verbatim}, `Returns`{.verbatim},
    `Examples`{.verbatim}) for scientific code.
-   `Sphinx`{.verbatim} with `autoapi`{.verbatim} generates a complete
    API reference by scanning your source code, requiring no manual
    `.rst`{.verbatim} files per module.
-   `intersphinx`{.verbatim} enables cross-project links (e.g., to
    NumPy, Python), making your documentation part of the broader
    ecosystem.
-   Documentation builds can be automated with GitHub Actions and
    deployed to GitHub Pages on every push to `main`{.verbatim}.
-   **PR Previews** let reviewers see documentation changes visually
    before merging, catching formatting issues that a green checkmark
    cannot.
:::
