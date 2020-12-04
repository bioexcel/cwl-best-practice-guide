---
title: Finding / creating tools
sort: 3
---

# Finding / creating tools

CWL describes only the workflow for your process. To run this process you will need to use suitable tools, for which you will need to write, or use pre-existing, CWL descriptions.

## Locate existing CWL-described tools

Before writing a CWL interface for a tool, it is recommended you check to make sure no-one else has done so. Libraries of tools which already have CWL interfaces are available for general use. For example:
* <https://github.com/common-workflow-library/bio-cwl-tools> is a library of CWL-specific CommandLineTool descriptions for biology/life-sciences related applications
* <https://dockstore.org/> is a library of docker based tools described using either CWL, Workflow Description Language (WDL), Nextflow, or Galaxy
* <https://github.com/bioexcel/biobb_adapters> is a library of adaptors (including for CWL) for popular bioinformatic tools


### versioning of CWL tool descriptions 

Copying CWL interface code, when you find it, directly into your own workflow is not recommended. Doing this will break your link to the original tool, so when that is updated you will have to update your copy (if you notice).

Instead it is better, where possible, to link to, or import directly, the original CWL tool description. If the CWL interface is provided as a library (installed via pip or conda, for example) then it can be installed this way. Alternatively, if it is available within a git repository, you can use `git submodules` to link to that repository from yours.


## Creating your own CWL-described tools

If a CWL description does not exist for your tool of interest, then you will need to create it. These can be installed and run within your local environment however, to ensure portability it is recommended to create a container image for the tool, and access that using CWL.

### Find existing container images

You can search for existing containers at <https://hub.docker.com/> and <https://quay.io/> (includes BioContainer). Make sure to check the origin of the container, and also check to make sure the tool version is up to date.

### Finding existing packages in BioConda/BioContainers

Check <https://bioconda.github.io/> and <https://biocontainers.pro/> for suitable packages


### Check license requirements of tools

Consider the use that you will be putting the tool too, and check the license of the tool to make sure it is suitable.

### Did someone already write a CWL description?

Check global GitHub search for Common Workflow Language files as last resort. If you do find an interface which hasn't been added to the above libraries, and it is publicly available and appropriately licensed, then suggest to the developers that they might want to submit the interface to one (or more) of the libraries.


## Further reading

 * `git submodules`
  * Using `biobb_adapters` as example for linking to a tool library: <https://github.com/bioexcel/biobb-wf-md-setup-protein-cwl>


