+++
title = "Coa Programming Language"
tags = ["programming"]
+++

Authors: Ken Shibata, [Shinya Takamaeda](https://sites.google.com/site/shinyaty/home)

Mainstream programming languages often require special syntax to improve execution speed through parallel execution, and it is not easy to achieve parallel execution. We have developed a parallel-oriented programming language "Coa" with parallel execution as the default. To avoid race conditions, Coa automatically detects variable dependencies and executes programs in parallel, similar to out-of-order execution of CPUs. Therefore, the behavior from the outside is the same as sequential execution, and it is expected to improve execution speed without increasing program complexity.

[GitHub](https://github.com/nyiyui/go-coa)
&bull; [Manuscript](/projects/coa/coa_swopp-2022.pdf)
&bull; [Slides](https://docs.google.com/presentation/d/1Ooln_pojSlW_T7ZXb0aWG5PyNMTQwvMJoWqgsoFvu0U/preview)
&bull; [Presentation Video (in Japanese)](https://youtube.com/watch?v=fqGx2pAEzeg)
