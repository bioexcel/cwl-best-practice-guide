---
title: Docker containers
sort: 1
---

# Docker containers

This section explains further the [interoperability](interoperability.md)
recommendation to use Docker containers to execute tools from CWL workflows.

## What are containers?

In short, a _container_ is a set of processes 
executed as an isolated part of the operating system, with its own
network and file system hierarchy; that is `/` and `localhost` inside the container
is separate from the host. The file system is initialized from a 
_container image_ which typically have a miniature Linux distribution
and the required binaries pre-installed along with their dependencies. 

```info
Containers are implemented by a set of 
[Linux kernel features](https://linuxcontainers.org/) and can be considered an evolution
of [Solaris Zones](https://docs.oracle.com/cd/E18440_01/doc.111/e18415/chapter_zones.htm)
and [FreeBSD Jails](https://www.freebsd.org/doc/handbook/jails.html) although with different focus.
```

The most popular container technology is [Docker](https://www.docker.com/), which adds
command line tools and daemons to manage and build containers, as well as
download ("pull") pre-built container images from online repositories. 

In _dev-ops_ environments Docker is often used as a way to combine 
networked _microservices_ (one per container) or to deploy long-running
server applications (as a light-weight VM alternative); however for CWL 
[CommandLineTool](https://www.commonwl.org/v1.2/CommandLineTool.html)
containers are used for individual executions of 
command line tools that exchange files. In CWL you'll typically wrap
each tool binary as a separate container image, 
and each step invocation creates a new temporary container that is
removed after the workflow finishes.

```tip
It is possible to build your own Docker images using a reproducible
[Dockerfile](https://docs.docker.com/engine/reference/builder/) 
recipe with shell script commands for file system 
changes to add on top of a _base image_. 
The built image can be kept locally, or deposited in a 
repository like [Docker Hub](https://hub.docker.com/) or
[Quay.io](https://quay.io/).
```

## Containers in workflows

For workflows the use of containers provides a consistent runtime environment 
for individual tools, ensuring that all required dependencies and configurations
are included and in predictable paths. Containers also provide a level of isolation, 
which mean that your workflow can combine tools that could otherwise have 
conflicting dependency requirements. 

```info
On macOS and Windows desktops, containers transparently
execute tools in a Linux Virtual Machine, ensuring workflow tools are executed
in the same environment across host operating systems. Care should be taken to
ensure the VM has sufficient memory required by the workflow's tools.
```

## How the workflow engine use containers

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

Although workflows can mix container and non-container steps, it is recommended to 
transition all steps to use containers so that the whole workflow is portable.

[cwltool](https://github.com/common-workflow-language/cwltool#using-user-space-replacements-for-docker) 
can take an option `--default-container debian:9` which ensures all command line tools run in a container 
with the given base image. It is recommended to transition workflows to use explicit 
images with `DockerRequirement` even for "regular" POSIX tools like `grep` and `sed`.

### Docker in Docker?

Note that normally the CWL engine itself does **not** run in a container,
as it is responsible for coordinating the workflow and the other containers
by calling the  `docker` command line.

```warning
If a tool use containers internally, it can be tricky or insecure to 
execute that tool from an outer container, as Docker-in-Docker requires
[privileged mode](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)
which in effect is giving away full `root` access to the container.
```

Although it may be possible to achieve nested containers in a more secure way
using [Singularity](https://sylabs.io/docs/), 
CWL's [DockerRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#DockerRequirement)
do not have currently have support for privilege options.

## Finding container images

See [finding tools](tools.md) for how to find existing container 
images for many bioinformatics tools.

The below CWL example assumes we want to run <https://hub.docker.com/r/curlimages/curl> from the Docker Hub, which image name corresponds to `curlimages/curl`:

```yaml
hints:
  DockerRequirement:
    dockerPull: curlimages/curl
```

For using images from other repositories like <https://quay.io/>, the `dockerPull` must be qualified with a hostname:

```cwl
hints:
  DockerRequirement:
    dockerPull: quay.io/opencloudio/curl
```

It is recommended to use the official Docker image from a tool's project when it's available, although there can be many reasons to pick a different image:

* Container has not been updated for latest release
* Desired plugins or compile options were not enabled
* Upstream container image is unnecessarily large, an alternative based on [alpine](https://hub.docker.com/_/alpine) and [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) might reduce the Docker image size
* Upstream image does not have desired hardware optimization, e.g. 


It is not recommended to reference private Docker repositories or use the 
[DockerRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#DockerRequirement)
options of `dockerLoad`/`dockerImport`, except for proprietary software which can't be distributed in public repositories. 

Similarly the `dockerPull` option should only be used with a local `Dockerfile` if customizations are needed in order to run the tool from CWL. If a desired open source tool do not exist as a container image, you can use `dockerFile` as an experiment before then publishing the image to Docker Hub under your user or organization's own namespace, replacing `dockerFile` with the updated referencing in `dockerPull`:

```cwl
hints:
  DockerRequirement:
    #dockerFile: curl-Dockerfile
    dockerPull: stain/curl
```


## Which container engine?

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

```warning
The [Windows 10 Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
can also [support Docker](https://docs.docker.com/docker-for-windows/wsl/) but this configuration
is currently not recommended for CWL users; bioinformatics tools in such containers 
may crash due to subtle differences between Linux kernel and the WSL2 subsystem, 
particularly for POSIX file system access.
```

## Alternative container engines

All major CWL engines support Docker, but some of them also support 
[alternative container systems](https://docs.dockstore.org/en/develop/advanced-topics/docker-alternatives.html), 
although their configuration vary slightly:
    
* [cwltool](https://github.com/common-workflow-language/cwltool#using-user-space-replacements-for-docker): Docker by default, or `cwltool --singularity`
  or  `cwltool --user-space-docker-cmd=udocker` or `cwltool --user-space-docker-cmd=nvidia-docker`
* [Toil](https://toil.readthedocs.io/en/latest/running/cwl.html): Docker by default, or `toil-cwl-runner --singularity`
* [Cromwell](https://cromwell.readthedocs.io/en/stable/tutorials/Containers/#singularity): Docker by default, Singularity through configuration
* [Arvados](https://doc.arvados.org/v2.1/install/install-docker.html): Only Docker supported
* [CWL-Airflow](https://cwl-airflow.readthedocs.io/): Only Docker supported
* [REANA](http://docs.reana.io/): Only Docker supported

```note
[Singularity](https://sylabs.io/docs/) is popular in HPC centres as it permits more 
fine-grained access control and is considered more secure than Docker. 
Singularity can use its own container images or load 
Docker images; however with CWL [DockerRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#DockerRequirement) 
only loading of Docker images are supported.
```

[nvidia-docker](https://github.com/NVIDIA/nvidia-docker) is a wrapper of `docker` that handles bindings and permissions for NVIDIA GPUs. It can be used together with CUDA-optimized containers like [gromacs/gromacs](https://hub.docker.com/r/gromacs/gromacs/).

```note
[udocker](https://indigo-dc.gitbook.io/udocker/) is a lighweight user-space alternative to Docker, 
with a largely compatible command line interface. By not requiring `root` privileges it can
be easier to install, but instead of using Linux kernel features for containers
this instead rewrites library calls to pretend the 
```

An alternative to containers that do not require `root` privileges 
is to use [Conda packages](conda.md).

## Tips and caveats

When [executing on cloud nodes](https://toil.readthedocs.io/en/latest/gettingStarted/quickStart.html#awscwl) or
on a local cluster, the chosen container technology will need to 
be installed on each node or be part of the node's base image. 
It is recommended to keep the same version on the workflow head node as on the compute nodes. 

Even though a workflow does not have any `DockerRequirement`s,
containers may be used by the CWL engine to evaluate
[Javascript expressions](https://www.commonwl.org/user_guide/13-expressions/index.html) if no
compatible version of `node` is installed.
