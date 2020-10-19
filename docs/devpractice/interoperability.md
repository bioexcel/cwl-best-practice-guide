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



## Avoiding off-workflow data flows

## Parameterizing a workflow

## Adding type annotations

