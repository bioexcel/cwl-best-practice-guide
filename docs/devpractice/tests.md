---
title: Writing incremental tests
sort: 4
---

# Writing incremental tests

Providing tests for workflows is important - they enable new users to be sure that a workflow behaves as expected, and also gives developers a reference to use to ensure that any changes made to the tools doesn't break the workflow. Setting up tests can be daunting, and time consuming at the start, but doing so will pay off in the long term. Below we discuss some of the details of how to do this.

## Find/create test data for each step

If the data is open then partial data from earlier steps is easiest to use.

Intermediate data for a workflow can be generated by calling the CWL reference implementation `cwl-runner` with the `--cachedir` option. This will cache all step outputs, which you can then copy to create test data for distribution. A perhaps more tedious alternative is to add workflow level `outputs` mapped from every step's outputs and copy test data manually.

The `--cachedir` option, sometimes combined with `--target` is also useful for incremental testing of a workflow, as it will reuse old outputs as long as a tool configuration and tool inputs match exactly a previous execution.Note that this means the step outputs are assumed to be reusable - for tools with mutable state (e.g. `wget` of current weather or a database lookup) add the [WorkReuse](https://www.commonwl.org/v1.2/CommandLineTool.html#WorkReuse) hint with `enableReuse: false` to force re-execution.

## Structure workflow files to support testing

CWL workflows can be [nested](https://www.commonwl.org/user_guide/22-nested-workflows/index.html). When writing a complex workflow it can be better to break this into several scripts, so that each can be tested independently.

This also allows you to make siblings of the parent workflow to tweak data handling, while retaining most of the reusable steps inside the nested workflows.


## Setting up automated testing with GitHub Actions

If you are using GitHub to host your git repository, then you can make use of [GitHub Actions](https://docs.github.com/en/actions) to automate your testing process. Setting up the tests can be a (relatively) straightforward process, if you follow a few simple rules, and approach this in an iterative manner.

The minimum requirements are:
- GitHub Actions script
- package dependency file for your script

Your _dependency file_ will describe the packages needed to run CWL (in the first instance - you can add other dependencies later). If you use conda for installing the `cwl_runner` then this could be as simple as:

```yaml
name: cwlrunner
channels:
  - conda-forge
  - defaults
dependencies:
  - cwl_runner
```

This should be saved in your git repository - in this example the file will be called `env.yml`.

The GitHub actions script should be saved in a folder `.github/workflows/` within your git repository. The basic layout this takes is:

```yaml
name: CI testing

on: [push, pull_request]

jobs:

  workflow_validation:
  
    runs-on: ubuntu-latest
    
    steps:
    
    - name: Checkout repository
      uses: actions/checkout@v2
    
    - name: Set up Conda
      uses: conda-incubator/setup-miniconda@v2
      with:
        auto-update-conda: true
        python-version: 3.8
        activate-environment: cwlrunner
        environment-file: env.yml
     
     - name: Validate Script
       run: |
         conda run -n cwlrunner cwltool --validate script.cwl
```

This will validate the `script.cwl` CWL script when a `push` is made to the GitHub repository, or when a `pull_request` is made to the repository. 

The first two steps, `Checkout repository` and `Set up Conda`, use the [checkout](https://github.com/actions/checkout) and [setup-miniconda](https://github.com/conda-incubator/setup-miniconda) actions from the GitHub marketplace (these are, generally, a lot simpler to use than writing your own scripts, and should work on most available testing OS). 

The last step, `Validate Script`, runs a simple shell command (the shell is not a login shell, so conda cannot be activated normally and `conda run -n [env]` must be used instead) that carries out the validation of the workflow script.

```tip
Just like in computational workflows, it is important to [lock down versions](interoperability.html#locking-down-versions-of-command-line-tools)
of GitHub Actions you rely in, as shown above with `@v2` and `python-version`.
```


If your repository uses [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) for loading libraries, then these can be loaded by adapting the _checkout_ step to enable `submodules`:

```yaml
    - name: Checkout repository
      uses: actions/checkout@v2
      with:
        submodules: recursive
```


Once you have a script which successfully runs the validation test (even if not all steps validate successfully), you can start adding other tests as extra steps in the script. These will require inputs for your CWL workflow, which can be gathered as described above, and stored in a `tests/` directory, along with the configuration file for the tests. These can be added as extra steps in your script as they are provided, e.g.:

```yaml
     - name: Run Testcase Script
       run: |
         conda run -n cwlrunner cwltool script.cwl tests/testcase.yml
```



## Defensive CWL programming

It is good practice to specify a `format` for all `File` objects passed into / out from your tools. These should, where possible, refer to an existing formal vocabulary description, such as the [EDAM PDB format](http://edamontology.org/format_1476), e.g.:
```
inputs:
  input_file:
    type: File
    format: http://edamontology.org/format_1476
```
or (if your script uses namespaces):
```
$namespaces:
  edam: http://edamontology.org/
inputs:
  input_file:
    type: File
    format: edam:format_1476
```

Doing this will provide a prompt for the user to check the format of their input file. Note that the file itself is not checked by the cwltool, only the workflow annotation, so it is of course possible for a user to simply label their incorrect input file as if it was the correct format, in order to try to run the workflow. But adding this `format` check will assist them in debugging their issues once they see that this does not work.

You can specify your own, local, format for the `File` objects - this allows for quick development of new tools and workflows, at the cost of some traceability. If you do this it is recommended to include format information in the documentation that you provide with the tool and, in the long-term, to submit your format for inclusion in an ontology or vocabulary (such as EDAM).
