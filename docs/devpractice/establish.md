---
title: Establish workflow development practices
sort: 1
---

# Establish workflow development practices

CWL workflows are managed in files, so treat them like any other source code: Use revision control (e.g. git), versioning, testing, license, documentation, README. 

## Development Philosophy

Workflow development is best approached in an incremental development manner. Once a workflow has been planned out, then the task of building it should be split into discrete parts, each of which is self-contained, with well defined inputs and outputs. 



## Version control

Version control is important for software development, of which workflow development is a part, as it enables better collaboration with co-workers, the ability to track changes, and to label working versions of the code (so these are easy to find again). Probably the most commonly used system at the moment is git (see below for an introduction to using git), which is a distributed version control system. This can be used locally on your own machine, but you can also use services such as github or gitlab to host copies of your repositories, enabling easier collaboration with others.

<!-- 
include here a schematic of standard layout of a git repository? 
** source file
** test suite directory
** README file
** License file
 -->
 
As development progresses, it is common to tag versions of the code which are intended for release for others to use. The standards for this are:
* 0.X.X
 * While in development releases should be prefixed with 0, to show they may not be stable
* 1.X.X
 * Once the workflow is considered 'finished' then a 1.0 release can be made. This does not mean all bugs will be fixed, or that changes might not be needed. Any of these that are released will come as .X or .X.X releases.
 * If a release is made which introduces new features, then it is common to increment the major number (i.e. 1.X to 2.0).

This tagging can be done within git, and both github and gitlab are designed for highlighting these tagged releases to make it easier for users to find the version of your code that you want them to find.


## Testing

It is good practice to include tests with your code, so that developers and users can be confident it is behaving as expected (this is important both when developing new features, as well as using the code on new systems).

CWL does not have a test suite in the same manner that one is available for python (for example). However, what you can do is to create standard inputs for individual sections of your workflow, and retain the outputs from a known working version of each section. These can then be used as benchmarks for others to use in their workflow, to ensure it produces the same results.

<!--
Are we able to run single steps in a workflow, and compare with standard outputs? I think testing will be a lot more manual process for CWL than for other languages?
-->

## Workspace Structure

To organise your working and testing code we recommend adopting this structure to your git repository:
```
📦workflow_git_repository
 ┣ 📂biobb@
 ┣ 📂tools
 ┃ ┣ 📜tool1.cwl
 ┃ ┗ 📜tool2.cwl
 ┣ 📂test
 ┃ ┣ 📂tool1
 ┃ ┃ ┣ 📂tool1.prov
 ┃ ┃ ┣ 📜tool1_test.yml
 ┃ ┃ ┣ 📜tool1_test.input
 ┃ ┃ ┣ 📜tool1_test.output
 ┃ ┗ 📂tool2
 ┃   ┣ 📂tool2.prov
 ┃   ┣ 📜tool2_test.yml
 ┃   ┣ 📜tool2_test.input
 ┃   ┗ 📜tool2_test.output
 ┣ 📜workflow.cwl
 ┗ 📜workflow_settings.yml
```
The main workflow script (and example configuration yaml file) would be in the root directory, with tools in a subdirectory (and tools in other repositories linked using git submodules). The test directory will contain a directory for each tool requiring testing. Each of these directories will contain example input / output data, a YAML configuration file for running the test, and a directory containing provenance information for these tests. How to create and use these testing tools will be described in later sections.

## Documentation

### README

A README.md file should always be included with a project. This should cover these basic topics:

* What does this workflow do?
* What (software) does it need to run?
* How do you install it?
* How do you use it?
* Attribution, and contact details
* License

<!--
should we include here a section on constructing fuller documentation - using read the docs (or similar software)?
-->


## License

A license should be included with your code, to help users to work out how they can use your code. You should mention the license chosen in your README file (see above), and also include a copy of the license text in your repository. Advice on choosing a license to suit your needs can be found here: <https://choosealicense.com/>. If you are not sure what may be the best for you, then consult with other developers in your area to see what license they use for their work, as often using the same license will help on collaborative projects.



See:
* <https://swcarpentry.github.io/git-novice/>
* <https://doi.org/10.1371%2Fjournal.pcbi.1005412>
* <https://doi.org/10.1371/journal.pcbi.1005265>

