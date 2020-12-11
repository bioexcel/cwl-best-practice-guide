---
title: Publishing in workflow repositories, documentation
sort: 11
---

# Publishing in workflow repositories, documentation

While you are developing your workflow, you should keep in mind future users who have to learn to use your workflow, both others and yourself (as returning to your work after some period is almost the same as opening for the first time code written by a complete stranger). It is important to write documentation for the workflow as you go, to facilitate this, as well as considering where you will archive or publish the workflow.

## Workflow documentation

Most objects in CWL have `label` and `doc` fields. The `label` field should be used for a short, human readable, label for the object; the `doc` field takes strings, or an array of strings, and should contain documentation on what the object is and how to use it. Getting into the habit of filling these in as you build your workflow makes documentation writing a more natural part of the development process. This is far less painful than writing the full documentation for everything after your workflow development has been finished.

## Workflow visualisation

The reference implementation of CWL, `cwltool`, has the `--print-dot` option, which prints a file suitable for processing with the Graphviz `dot` program (which will be installed alongside cwltool) to generate a Scalable Vector Graphic (SVG) file. The bash oneliner to do this is:
```bash
cwltool --print-dot my-wf.cwl | dot -Tsvg > my-wf.svg
```
Including a graphic visualisation of your workflow in your documentation will help users to quickly grasp the dependencies of each step, and what the likely running order of the workflow will be.

## GitHub 

incl. <https://github.com/common-workflow-library>

## Workflow Hub 

<https://workflowhub.eu/>

## Dockstore

<https://dockstore.org/>

## BioBB workflow example 

incl <https://biobb-wf-md-setup.readthedocs.io/>
