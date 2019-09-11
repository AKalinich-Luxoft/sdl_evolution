# Using cppcheck and PVS-studio in Open Source

* Proposal: [SDL-NNNN](NNNN-pvs-cppcheck-usage.md)
* Author: [Andrii Kalinich](https://github.com/AKalinich-Luxoft)
* Status: **Not created**
* Review manager: N/A
* Impacted Platforms: [SDL core]

## Introduction

This proposal will be to introduce usage of [Cppcheck](https://sourceforge.net/p/cppcheck/wiki/Home) and [PVS-Studio](https://www.viva64.com/en/pvs-studio) static analyzers within OpenSDL.

## Motivation

Usage of allows to detect potential bugs, vulnerabilities, undefined behavior etc at the compile time. Of course, this may also be achieved through manual code reviews. But using automated static analysis tools is much more effective.

For example Cppcheck provides unique code analysis to detect bugs and focuses on detecting undefined behaviour and dangerous coding constructs. The goal is to detect only real errors in the code (i.e. have very few false positives).

PVS-Studio is a tool for detecting bugs and security weaknesses in the source code of programs, written in C, C++, C# and Java. It works under 64-bit systems in Windows, Linux and macOS environments, and can analyze source code intended for 32-bit, 64-bit and embedded ARM platforms.

## Proposed solution

Static analysis tools can be run locally on developers workstations. But for a better and wider automatization it is nice to use these tools in conjunction with the [OpenSDL CI server](http://opensdl-jenkins.luxoft.com:8080/)

## Detailed design

Proposed solution is to create a new job for a static analysis which will be able to take SDL source code from the particular branch, analyze it using cppcheck and pvs-studio tools and generate HTML report with details regarding found issues. This HTML report should be attached to each CI job and available to view/download so each developer may quickly look and analyze all found issues.
Optionally, it's also nice to have a possibility to analyze only code changed by developer (for example only diff if it's a PR). In that case developer will see only issues caused by his code and not all issues existing in the mainline of the project.

## Impact on existing code

No impact on existing SDL code as changes are related to CI configuration only.

## Alternatives considered

Static analysis tools may be used locally by each developer but in this case it's a possible discrepancy between local analysis and CI analysis due to different tools versions, environment etc.
