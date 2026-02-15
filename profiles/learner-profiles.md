---
title: "Learner Profiles"
---

# Ana: The Grad Student with a Growing Codebase

Ana is a second-year PhD student in computational chemistry. She writes
Python scripts daily to process simulation data and generate plots. Her
`analysis/`{.verbatim} folder has grown to dozens of files, and she
frequently copies functions between projects. She has heard of
`pip install`{.verbatim} but has never created her own package. She
wants to organize her code so that her labmates can use it without
needing to clone her entire project directory.

Ana will benefit most from the episodes on **modules**, **pyproject.toml
with uv**, and **quality assurance**. She needs to learn how to turn her
collection of scripts into an installable package and add basic tests so
her labmates can trust the code.

# Raj: The Postdoc Who Needs Reproducibility

Raj is a postdoc in materials science. He has published two papers where
reviewers asked for the analysis code, and each time he spent days
getting scripts to run on a clean machine. His code depends on
`numpy`{.verbatim}, `scipy`{.verbatim}, and a Fortran library he
inherited from a former group member. He is comfortable with Python but
has never written a `pyproject.toml`{.verbatim} or used a build system.

Raj will benefit from the full lesson arc, especially **reproducible
scripts**, **release engineering**, and **compiled extensions**. He
needs to learn how to declare dependencies, pin versions, and build
binary wheels that include his Fortran code.

# Sofia: The RSE Supporting a Research Group

Sofia is a Research Software Engineer embedded in a physics department.
She maintains three internal Python packages and helps researchers
publish their code. She is familiar with `setuptools`{.verbatim} and
`pip`{.verbatim} but wants to modernize the group's workflow. She has
heard about `uv`{.verbatim} and `meson-python`{.verbatim} but has not
tried them yet.

Sofia will use this lesson as a reference for migrating existing
projects. The episodes on **pyproject.toml with uv**, **quality
assurance**, **release engineering**, and **compiled extensions** are
directly relevant to her daily work.

# Liam: The Master's Student Starting From Scratch

Liam is a first-year Master's student in bioinformatics. He has taken
one Python course and can write basic scripts with `pandas`{.verbatim}.
His advisor has asked him to "make his analysis reproducible" and he is
not sure where to start. He has never used the terminal beyond running
`python script.py`{.verbatim}.

Liam should start from the very beginning of the lesson. The episodes on
**writing reproducible Python** and **modules** will give him the
conceptual foundation he needs before moving on to packaging and
tooling.

# Elena: The PI Who Wants Better Group Practices

Elena is a tenure-track professor who leads a computational materials
science group of eight students and postdocs. She has noticed that
onboarding new members takes weeks because each project has its own
ad-hoc setup, and published results are difficult to reproduce a year
later. She can write Python but delegates most coding to her team.

Elena will not attend the full workshop but needs the **instructor
notes** and **setup** materials to evaluate whether to adopt these
practices group-wide. The episodes on **quality assurance**, **release
engineering**, and **CI/CD** are most relevant to her goal of
establishing reproducible lab standards.

# Marco: The Domain Scientist with Legacy Fortran

Marco is a senior PhD student in theoretical chemistry. His group
maintains a Fortran codebase (\~15k lines) for electronic structure
calculations that dates back to the early 2000s. He has written Python
wrappers using `ctypes`{.verbatim} and `subprocess`{.verbatim} calls,
but the build process is brittle and only works on the group's cluster.
He wants to make the code installable via `pip`{.verbatim} so
collaborators can use it.

Marco will benefit most from the episodes on **compiled extensions** and
**release engineering**. The Meson build system and
`cibuildwheel`{.verbatim} pipeline will directly address his
distribution challenge. The **documentation** and **CI/CD** episodes are
also relevant for maintaining the long-term health of his project.
