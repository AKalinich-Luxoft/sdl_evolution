# Dynamic analysis - Address and memory sanitazing 

* Proposal: [SDL-NNNN](NNNN-address_and_memory_sanitizing.md)
* Author: [Andrii Kalinich](https://github.com/AKalinich-Luxoft)
* Status: **Not created**
* Review manager: N/A
* Impacted Platforms: [SDL core]

## Introduction

This proposal will introduce usage of [address and memory sanitizing tools](https://github.com/google/sanitizers) within OpenSDL.

## Motivation

Dynamic analysis is the testing and evaluation of a program by executing data in real-time. The objective is to find errors in a program while it is running, rather than by repeatedly examining the code offline.
By debugging a program in all the scenarios for which it is designed, dynamic analysis eliminates the need to artificially create situations likely to produce errors. Other advantages include reducing the cost of testing and maintenance, identifying and eliminating unnecessary program components, and ensuring that the program being tested is compatible with other programs.
Integration into the development process of SDL with address and memory sanitizing will effectively reduce the number of crashes in production and make whole product much more stable.

## Proposed solution

Dynamic analysis tools can be run locally on developers workstations. But for a better and wider automatization it is nice to use these tools in conjunction with the [OpenSDL CI server](http://opensdl-jenkins.luxoft.com:8080/)

## Detailed design

Proposed solution is to create a new job for a dynamic analysis which will be able to take SDL source code from the particular branch, build SDL binaries with dynamic analyzers included, run built SDL and emulate some common test scenarios, stop SDL and generate report with found issues. This report should be attached to each CI job and available to view/download so each developer may quickly look and analyze all found issues.

Can be created a few different jobs depending on type of analysis. For example Google provides:
- [Address sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) - memory error detector for C/C+
- [Leak sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer) - memory leak detector which is integrated into AddressSanitizer
- [Memory sanitizer](https://github.com/google/sanitizers/wiki/MemorySanitizer) - detector of uninitialized memory reads in C/C++ programs
- [Undefined Behavior Sanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) - fast undefined behavior detector

Optionally, it's also nice to have a possibility to analyze only code changed by developer (for example only diff if it's a PR). In that case developer will see only issues caused by his code excluding all issues existing in the mainline of the project.

## Impact on existing code

No impact on existing SDL code as changes are related to CI configuration only.

## Alternatives considered

Dynamic analysis tools may be used locally by each developer but in this case it's a possible discrepancy between local analysis and CI analysis due to different tools versions, environment etc.
