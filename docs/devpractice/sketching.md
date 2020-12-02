---
title: Sketching out workflow
sort: 2
---

# Sketching out workflow

## Is there a baseline to start from?
Based on existing shell scripts or workflow in other language?
..or published best-practice pipeline (e.g. GATK4)?

Do any similar open source workflows exists (even in another language)?

Consult:
* <https://www.myexperiment.org/home>
* <https://dockstore.org/> 
* <https://workflowhub.eu/>
* <https://nf-co.re/>
* ..

## Start with high level organisation

CWL workflows can be nested - take advantage of this by constructing a relatively abstract workflow, with high-level functions. Within these you can fill in details with nested steps as required.

## What will be the inputs and outputs data and formats? 

What are their storage/access requirements? And what formats will these have, can you control these with any entomology definitions?


## What will be (if any) the intermediate data and formats?

CWL steps operate in discrete environments, and any data from one step needed for the next has to be explicitly specified in the workflow. So in your sketch you should include the requisite the intermediate files produced by a step which are needed by the next.


# What are the execution restrictions?

Can it run on the public cloud? Is data confidential? Cost/time limits?
Can workflow be developed in open?

# Making a valid non-executable workflow

Version 1.2 of CWL introduces the [Operation](https://www.commonwl.org/v1.2/Workflow.html#Operation) class. This can be used to represent a potential step in a workflow which does not yet have a CommandLineTool, Workflow or ExpressionTool implementation. It allows for a workflow to be created, which will not be executable, but will be valid for the purposes of running analysis of the workflow, such as printing RDF graphs, or workflow visualization.

The use of this class is described further in [Writing Partial Workflows](https://docs.bioexcel.eu/cwl-best-practice-guide/devpractice/partial.html#using-abstract-operations-as-placeholders).


