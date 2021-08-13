---
title: Publishing in workflow repositories, documentation
sort: 11
---

# Publishing in workflow repositories, documentation

When publishing your workflow you should keep in mind the [FAIR](https://fair-dom.org/knowledgehub/data-and-model-management/) principles, and make sure that your workflow is both **Findable** and **Accessible**.

To ensure your workflow is **Findable** you need to consider where you are publishing it. A personal git repository (perhaps hosted on github or gitlab) is a good tool for development work, but can be awkward for others to find without signposting. To help with this you can make use of one (or more) different workflow databases or libraries which are available on the web. A few of these will be highlighted below.

To ensure your workflow is **Accesible** while you are developing it, you should keep in mind future users who have to learn to use your workflow. This will include both others and yourself, as returning to your work after some period is almost the same as opening for the first time code written by a complete stranger. It is important to write documentation for the workflow as you go, to facilitate this, as well as considering where you will archive or publish the workflow.

## Workflow documentation

Most objects in CWL have `label` and `doc` fields. The `label` field should be used for a short, human readable, label for the object; the `doc` field takes strings, or an array of strings, and should contain documentation on what the object is and how to use it. Getting into the habit of filling these in as you build your workflow makes documentation writing a more natural part of the development process. This is far less painful than writing the full documentation for everything after your workflow development has been finished.

An example of good practice for this documentation can be taken from the [mdrun CWL tool interface](https://github.com/bioexcel/biobb_adapters/blob/master/biobb_adapters/cwl/biobb_md/mdrun.cwl):
```yaml
#!/usr/bin/env cwl-runner
cwlVersion: v1.0

class: CommandLineTool

label: Wrapper of the GROMACS mdrun module.

doc: |-
  MDRun is the main computational chemistry engine within GROMACS. 
  It performs Molecular Dynamics simulations, but it can also perform 
  Stochastic Dynamics, Energy Minimization, test particle insertion or 
  (re)calculation of energies.

baseCommand: mdrun

hints:
  DockerRequirement:
    dockerPull: https://quay.io/biocontainers/biobb_md:3.5.1--py_0

inputs:
  input_tpr_path:
    label: Path to the portable binary run input file TPR
    doc: |-
      Path to the portable binary run input file TPR
      Type: string
      File type: input
      Accepted formats: tpr
      Example file: https://github.com/bioexcel/biobb_md/raw/master/biobb_md/test/data/gromacs/mdrun.tpr
    type: File
    format:
    - edam:format_2333
    inputBinding:
      position: 1
      prefix: --input_tpr_path

$namespaces:
  edam: http://edamontology.org/

$schemas:
- https://raw.githubusercontent.com/edamontology/edamontology/master/EDAM_dev.owl
```
Note that the `label` and `doc` fields for the tool interface give an overview of the tool name and what it can be used for, while the `label` and `doc` fields for the `input` field (and repeated for the `output` fields in the linked file) describe what input is expected, and provides a link to an example file.

## Workflow visualisation

The reference implementation of CWL, `cwltool`, has the `--print-dot` option, which prints a file suitable for processing with the Graphviz `dot` program (which will be installed alongside cwltool) to generate a Scalable Vector Graphic (SVG) file. The bash oneliner to do this is:
```bash
cwltool --print-dot my-wf.cwl | dot -Tsvg > my-wf.svg
```
Including a graphic visualisation of your workflow in your documentation will help users to quickly grasp the dependencies of each step, and what the likely running order of the workflow will be.

## Workflow Libraries / Databases


### Common Workflow Library (GitHub) 

One of the most direct ways of sharing a CWL workflow or tool description will be to share it on github in the [Common Workflow Library](<https://github.com/common-workflow-library>). This contains the [Bio-CWL-tools](https://github.com/common-workflow-library/bio-cwl-tools) repository, which you could fork, add your CWL tool or workflow, and submit a pull request to the main repository again. This will make sure that anyone using this library will be able to find your tool, although at the expense of detaching the publication of your CWL interface from your development code if you have used your own local repository for that work. 

### Workflow Hub 

[WorkflowHub](<https://workflowhub.eu/>) is a workflow registry, with standardised workflow identifiers and metadata descriptions intended to make workflow discovery, reuse, and preservation more simple. It is workflow system agnostic, but uses CWL as the standardised language for workflow descriptors, so is particularly suitable for registering CWL workflows. Workflows are stored in the registry as a [Research Object Crate](https://www.researchobject.org/ro-crate/), and work is currently ongoing to integrate github access into the workflow upload process. If/when the registry reaches its end of service, then all published contributions will be made available through a public repository, such as Zenodo or Figshare, to ensure full data retention.

### Dockstore

[Dockstore](https://dockstore.org/) is a platform for sharing Docker-based tools described using a workflow language (specifically CWL, WDL, Galaxy or Nextflow). It uses integration with github to simplify workflow publication, reducing the friction of ensuring that the latest version of your workflow is published, but also provides local hosting of workflows if that does not fit in with your development cycle. Publication is orientated around github repositories, rather than individual workflows (as WorkflowHub is). Dockerstore also provide a [commandline interface (CLI)](https://docs.dockstore.org/en/develop/launch-with/launch.html#dockstore-cli) built on top of cwltool, allowing for easy access to hosted workflows.

### Zenodo / Figshare / Other

Instead of using a specialised workflow library you can publish directly on a general research archive system such as [Zenodo](https://zenodo.org/) or [Figshare](https://figshare.com/). Doing so will provide you with a [DOI](https://www.doi.org/) enabling others to reference your workflow easily, and the integration of Zenodo with github allows for simple updating of that record whenever you make a new release of your workflow. Searching Zenodo for [software archives which include the 'common language workflow' keywords](https://zenodo.org/search?q=common%20language%20workflow&type=software) shows this low-cost publication method is a relatively common practice. However these general research archives carry less information about your tool than the specialist archives above do, and so will require more work from end-users to find and access your tool.

## BioBB workflow example 

incl <https://biobb-wf-md-setup.readthedocs.io/>
