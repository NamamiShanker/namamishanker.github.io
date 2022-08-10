---
title: "Week 6: Continuing working on the frontend argparse"
date: 2022-06-15T21:34:26+05:30
draft: false
slug : ""
authors : ["Namami Shanker"]
tags : ["F2PY", "argparse", "cli"]
categories : ["F2PY", "Programming"]
externalLink : ""
series : ["GSoC'22"]
---

# Week 6: Continuing working on the frontend argparse

### What did I do this week?

I implemented some changes suggested by reviewers on my [draft PR](https://github.com/numpy/numpy/pull/21923).

Firstly, I added typing support to the functions I have made. Therefore, currently: `f2pyarg.py`, `service.py`, `utils.py` have typing support. Initially I had added typing library for type annotations. But [Bas Van Beek](https://github.com/numpy/numpy/pull/21923#discussion_r914597755) suggested to replace them with buitlins type annotations. These are not currently supported in Python versions below 3.9, but can be used by `from __future__ import annotations` statement. The `__future__` module is somewhat unique in that import therefrom enable behaviours that will only become default in future python versions. Older examples of this include the print functions and with statement, but a more recent addition is the postponed evaluation of annotations [(PEP 563)](https://peps.python.org/pep-0563/#abstract), which implicitly stringifies all annotations during runtime (e.g. a: int automatically becomes a: "int"). 

However, for Any type annotation I would still need to rely on typing module. Also, the `Optional` type annotation can also be replaced by builtins by piping the type with `None` keyword. (e.g. arr: Optional[list] can be replaced by arr: list | None). The tests work fine with the changes and the frontend looks more complete. The mid evaluation starts from the next week and I would need to present the changes to writers of F2PY. I hope my work is ready.

### What will I do next week?

I will fill out the mid evaluation form, and make changes according to the review.

### Did I get stuck anywhere?

No, I mostly applied changes requested in the review. These were mostly clear requests with helpful reviewers who clarified any doubts I had.