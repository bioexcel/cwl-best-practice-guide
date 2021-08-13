---
title: Finding / creating tools
sort: 3
---

# Finding / creating tools

CWL describes only the workflow for your process. To run this process you will need to use suitable tools, for which you will need to write, or use pre-existing, CWL descriptions.

## Finding CWL tool descriptons

Before writing a CWL interface for a tool, it is recommended you check to make sure no-one else has done so. Libraries of tools which already have CWL interfaces are available for general use. For example:
* <https://github.com/common-workflow-library/bio-cwl-tools> is a library of CWL-specific CommandLineTool descriptions for biology/life-sciences related applications (principally focused on bioinformatics).
* <https://dockstore.org/> is a library of docker based tools described using either CWL, Workflow Description Language (WDL), Nextflow, or Galaxy.
* <https://github.com/bioexcel/biobb_adapters> is a library of adaptors (including for CWL) for popular biomolecular simulation tools (including some machine learning).

### Using CWL tool descriptions.

Copying CWL interface code, when you find it, directly into your own workflow is not recommended. Doing this will make acknowledging the original developers more difficult, and will also break your link to the original tool, so if/when that is updated you will have to update your copy (if you notice at all that the original has been updated).

Instead it is better, where possible, to link to, or import directly, the original CWL tool description. If the CWL interface is provided as a library (installed via pip or conda, for example) then it can be installed this way. If you do this you can use the environmental variables `XDG_DATA_DIRS` and `XDG_DATA_HOME` to help your scripts to [discover CWL tool descriptions on your local filesystem](https://www.commonwl.org/v1.2/Workflow.html#Discovering_CWL_documents_on_a_local_filesystem). This would allow you to keep a library of CWL tool descriptions in a single location for use by multiple workflows. However you would need to make sure that you document how this is done, to ensure the portability of your scripts.

An alternative, if the CWL tool description is available within a git repository, is to use [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to link to that repository from yours. This will provide an explicit link to the tool from your workflow repository, making for clearer provenance of the CWL tool descriptions that you have used. An example of doing this for a BioExcel workflow is described [here](https://research-it.manchester.ac.uk/news/2020/09/28/advanced-git-linking-code-repositories-using-submodules/).

### Versioning of CWL tool descriptions 


## Creating CWL tool descriptions

If a CWL description does not exist for your tool of interest, then you will need to create it. These can be installed and run within your local environment. However, to ensure portability it is recommended to create a container image for the tool, and access that using CWL.

```tip
To find existing container images you can search for existing containers at [Docker](https://hub.docker.com/) and [Quay](https://quay.io/) (includes BioContainer). Make sure to check the origin of the container, and also check to make sure the tool version is up to date.
```

### Finding existing packages in BioConda/BioContainers

Check <https://bioconda.github.io/> and <https://biocontainers.pro/> for suitable packages


### Check license requirements of tools

Consider the use that you will be putting the tool to, and check the license of the tool to make sure it is suitable.

### Did someone already write a CWL description?

Check global GitHub search for Common Workflow Language files as last resort. If you do find an interface which hasn't been added to the above libraries, and it is publicly available and appropriately licensed, then suggest to the developers that they might want to submit the interface to one (or more) of the libraries.

### Design patterns for writing CWL tool descriptions

If you do find yourself needing to write the CWL tool description, then you can consult this [CWL pattern guide](https://github.com/common-workflow-library/cwl-patterns) to get some inspiration on what the best way to approach writing a tool description for your particular problem might be.


