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


## Using containers and Conda packages

CWL [CommandLineTool](https://www.commonwl.org/v1.2/CommandLineTool.html) steps can 
run any local shell commands, which is a good way to get started, for instance:

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



However such workflows are considered fragile as they:

* Rely on pre-installed tools (which may be outdated or unexpectedly become updated)
* Are hard to keep consistent on cloud execution nodes
* May use absolute paths, which needs to be changed on new systems
* Easily become platform-specific (e.g. macOS' BSD-based `sed` and Linux' GNU `sed` take different options)
* Do not document version of the tool or how to install it
* May rely on local files outside the workflow directory (e.g. `/etc` configurations)
* Add version constraints for common dependencies across tools (e.g. Python)

The easiest solution for CWL authors is often convert the step to 
[use Docker containers](https://www.commonwl.org/user_guide/07-containers/index.html). 

### What are containers?

In short, a _container_ is executed as an isolated part of the operating system, with its own
network and file system hierarchy, that is `/` and `localhost` inside the container
is separate from outside. The file system is initialized from a downloaded ("pulled")
_container image_ which typically have a miniature Linux distribution
and the required tools binaries pre-installed along with their dependencies. 

It is possible to build your own Docker images using a 
[Dockerfile](https://docs.docker.com/engine/reference/builder/) 
recipe with shell script commands to be added onto a _base image_. The built image can be kept
locally or deposited in a repository like [Docker Hub](https://hub.docker.com/) or
[Quay.io](https://quay.io/).

For workflows the use of containers therefore provides a consistent runtime environment 
for individual tools, ensuring that all required dependencies and configurations
are included and in predictable paths. Containers also provide a level of isolation, 
which mean that your workflow can combine tools that could otherwise have 
conflicting dependency requirements. 


```info
On macOS and Windows desktops, containers transparently
execute tools in a Linux Virtual Machine, ensuring workflow tools are executed
in the same environment across operating system. Care should be taken to
ensure the VM has sufficient memory required by the workflow's tools.
```

The CWL engine will handle staging of input/output files to the container, as 
long as they are formally declared 
(see [Avoiding off-workflow data flows](#avoiding-off-workflow-data-flows) below). 
The engine will transparantly compose the right `docker` commands, typically using 
[bind mounts](https://docs.docker.com/storage/bind-mounts/) to expose
only the required working directory of that particular step. 

```tip
Because the container is otherwise isolated from the host, the use of
containers also ensures that all file dependencies of the tool are either
included in the container image or declared as explicit input in CWL.
```

Note that normally the CWL engine itself does **not** run in a container,
as it is responsible for coordinating the workflow and the other containers. 
Although workflow can mix container and non-container steps, it is recommended to 
transition all steps to use containers. 

For testing and security purposes, 
[cwltool](https://github.com/common-workflow-language/cwltool#using-user-space-replacements-for-docker) 
can take an option `--default-container debian:9` which ensures all command line tools run in a container 
with the given base image.

```warning
If a tool use containers internally, it can be tricky or insecure to 
execute that tool from an outer container, as Docker-in-Docker requires
[privileged mode](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)
which in effect is giving away full `root` access to the container.

It may be possible to achieve this in a more secure way using [Singularity](https://sylabs.io/docs/), 
however CWL's [DockerRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#DockerRequirement)
do not have currently have support for privilege options.
```

### Finding the image

See [finding tools](tools.md) for how to find a good container image.

```yaml
hints:
  DockerRequirement:
    dockerPull: curlimages/curl
```

The parameter to `dockerPull` assumes public Docker images in Docker Hub; `curlimages/curl` corresponds to <https://hub.docker.com/r/curlimages/curl>

For using images from other repositories like <https://quay.io/>, the `dockerPull` must be qualified with a hostname:

```cwl
hints:
  DockerRequirement:
    dockerPull: quay.io/opencloudio/curl
```

It is not recommended to reference private Docker repositories or use the [DockerRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#DockerRequirement) options of `dockerLoad`/`dockerImport`, except for proprietary software which can't be distributed in public repositories. 

Similarly the `dockerPull` option should only be used with a local `Dockerfile` if customizations are needed in order to run the tool from CWL. If a desired open source tool do not exist as a container image, you can use `dockerFile` as an experiment before then publishing the image to Docker Hub under your user or organization's own namespace, replacing `dockerFile` with the updated referencing in `dockerPull`:

```cwl
hints:
  DockerRequirement:
    #dockerFile: curl-Dockerfile
    dockerPull: stain/curl
```



### BioConda

```yaml
hints:
  SoftwareRequirement:
    packages:
      curl:
        specs: 
          - https://anaconda.org/conda-forge/curl
```


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


### Docker or Singularity?

In order for the CWL engine to be able to execute tools in a container, the
execution node(s) will need to have a container system installed. The most
popular choice is the open source [Docker Engine](https://www.docker.com/products/container-runtime).

It is generally recommended to use the latest **stable** version of Docker Engine rather than a
version provided by the operating system, although with recent distributions either should work.

For Windows and macOS desktops, Docker also provide [Docker Desktop](https://www.docker.com/products/docker-desktop) 
which also helps manage the Virtual Machine. This install is a bit more extensive and includes
components that are not open source. The **stable** version is recommended for this use. 

```note
Docker on Windows 10 also supports running Windows OS inside containers, 
avoiding the need for virtual machines. While this is in theory would
work with CWL, it would make the CWL workflow incompatible with other
operating systems. 

It is not possible to run concurrently Linux and Windows containers on the 
same node, and most bioinformatics tools found as Docker images  
assume Linux x64 containers.
```

All major CWL engines support Docker, but

[Singularity](https://sylabs.io/docs/) is popular in HPC centres as it permits more fine-grained access control. Singularity can use its own container images or load Docker images; however from CWL only Docker images are supported.

[udocker](https://indigo-dc.gitbook.io/udocker/) is a 

but some of them also support 
[alternative container systems](https://docs.dockstore.org/en/develop/advanced-topics/docker-alternatives.html), 
although their configuration vary slightly:
    
* [cwltool](https://github.com/common-workflow-language/cwltool#using-user-space-replacements-for-docker): 
  - : `cwltool --singularity`
  - (which do not require `root` privileges but): `cwltool --user-space-docker-cmd=udocker`
* [Toil](https://toil.readthedocs.io/en/latest/running/cwl.html): Docker by default, or `toil-cwl-runner --singularity`
* [Cromwell](https://cromwell.readthedocs.io/en/stable/tutorials/Containers/#singularity): Docker by default, Singularity through configuration
* [Arvados](https://doc.arvados.org/v2.1/install/install-docker.html): Only Docker supported
* [CWL-Airflow](https://cwl-airflow.readthedocs.io/): Only Docker supported
* [REANA](http://docs.reana.io/): Only Docker supported

When [executing on cloud nodes](https://toil.readthedocs.io/en/latest/gettingStarted/quickStart.html#awscwl) or
on a local cluster, then the chosen container technology will usually need to 
be installed on each node or be part of the node's base image. 



## Avoiding off-workflow data flows

## Parameterizing a workflow

## Adding type annotations

