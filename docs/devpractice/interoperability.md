---
title: Ensuring workflow is portable and interoperable
sort: 7
---

# Ensuring workflow is portable and interoperable

An important aspect of CWL is _interoperability_ and _reproducibility_, 
allowing your workflow to run successfully not just on a different
machine or with a different CWL engine, but also survive the 
passage of time (avoiding _workflow decay_) 
and be _repurposable_ for different purposes.

In order to harden your workflow for future use, which, remember, might
include yourself, certain measures should be taken to give CWL
additional information about your tools, inputs and outputs.


## Use containers instead of local binaries

CWL [CommandLineTool](https://www.commonwl.org/v1.2/CommandLineTool.html) steps can 
run any local shell `baseCommand`, which is a good way to get started, for instance:

```yaml
baseCommand: "curl"
arguments: 
- "--location"
inputs:
  url:
    type: string
    doc: The URL of the resource to be downloaded. 
    inputBinding:
      position: 1
outputs:
  downloaded:
    type: stdout

```

The above CWL description can run the command line tool [curl](https://curl.haxx.se/), assuming it is available on `PATH`. It is also possible to use an absolute path for `baseCommand`, say to get a newer version:

```yaml
baseCommand: "/opt/bleeding-edge/bin/curl"
```

However such workflows are considered **fragile** as they:

* Rely on pre-installed tools (which may be outdated or unexpectedly become updated)
* Are hard to keep consistent on cloud execution nodes
* May use absolute paths, which needs to be changed on new systems
* Might be platform-specific (e.g. macOS' BSD-based `sed` and Linux' GNU `sed` take different options)
* Do not document version of the tool or how to install it
* May rely on local files outside the workflow directory (e.g. `/etc` configurations)
* Add version constraints for common dependencies across tools (e.g. Python 3.6 vs 3.8)

The easiest solution for CWL workflow authors is often to convert the steps to 
[use Docker containers](containers/docker.md) and/or [Conda packages](containers/conda.md):

```yaml
hints:
  SoftwareRequirement:
    packages:
      curl:
        specs:
          - https://anaconda.org/conda-forge/curl
  DockerRequirement:
    dockerPull: curlimages/curl
```

For further guidance, see the [Containers & packaging](containers/) section.

## Locking down versions of command line tools

One challenge with using external software packages is that a Docker image like [curlimages/curl](https://hub.docker.com/r/curlimages/curl) will resolve to the `latest` version, which over time will change. For instance at time of writing this version is `7.76.1`, but a later version of the tool may intentionally or unintentionally break functionality you rely on.

```info
A good tool will follow [Semantic Versioning](https://semver.org/) rules, meaning a version number `3.2.1` can be broken down as `MAJOR.MINOR.PATCH`, with upgrades roughly interpreted as:
* `PATCH` (e.g. `3.2.1` update of `3.2.0`) fixes bug in implementation, no new functionality
* `MINOR` (e.g. `3.2.0` update of `3.1.4`) adds functionality (e.g. new parameter), otherwise backwards compatible. May deprecate (but not remove) functionality, may change requirements (e.g. macOS > 10.14)
* `MAJOR` (e.g. `4.0.0` update of `3.2.1`) removes or changes existing functionality (e.g. remove parameter, different file format)

Unfortunately the reality is never as clean as this, and you should therefore ensure your workflow has sufficient [tests](tests.md) to help you verify an upgrade of a tool.
```

For provenance reasons it is therefore best practice to at least _document_ which version you have used/tested under `SoftwareRequirement`, and to lock down any `DockerRequirement` to a particular _tagged version_ of the image, e.g. `curlimages/curl:7.67.0`.

```yaml
hints:
  SoftwareRequirement:
    packages:
      curl:
        specs:
          - https://anaconda.org/conda-forge/curl
        version: [ "7.67.0", "7.71.1" ]
  DockerRequirement:
    dockerPull: curlimages/curl:7.67.0
```

Note above how `version` is an `[array]` indicating multiple versions. This may be useful if different operating systems provide varying versions, or where you want to indicate both the lowest and highest version you have tested.

```warning
A downside of locking down versions is that your workflow will not benefit from bugfixes of the tool. It is therefore important for production workflows that you try later versions of the tools, and verify workflow functionality using [tests](tests.md).
```



## Avoiding off-workflow data flows

Workflows ideally should document the whole process obtaining your final results. Any extra processes, such as downloading input files, and arranging or reformatting these, or moving/reformatting intermediate data, which are not included in your workflow script reduce the portability and reproducibility of your scripts. Best practice is to include all dataflows into the workflow script. If this is not possible (if, for example, certain inputs are not available for download from a public source, or the workflow is run in a restricted environment without access to these), then you will have to document the extra steps in the README file that accompanies the workflow.

<!-- ## Parameterizing a workflow TODO -->

## Adding type annotations

Where possible you should explicitly add type annotations to your file declarations. These will enable the cwltool to carry out basic checks on your input and output files, to ensure that they are what is expected. In the CWL tutorial there is further guidance on [specifying file formats](https://www.commonwl.org/user_guide/16-file-formats/index.html), as well as how to [create custom types](https://www.commonwl.org/user_guide/19-custom-types/index.html) for your file definitions. Creating custom types when you work with specialised data files will help ensure that new users of your workflow will be notified of errors in inputs that they might not necessarily have thought about checking.
