---
title: Sketching out workflow
sort: 2
---

# Sketching out workflow

When designing your workflow, there are a few steps you can take to make this process as painless as possible.

## Is there a baseline to start from?

Rather than designing a workflow from scratch, it is often more helpful to build up from an existing workflow. This could be because you are creating a workflow similar in design to (or a variant of) other processes that you already follow, or because you are converting a workflow from a collection of existing shell (or other language) scripts that you currently use. Even if neither of these are the case, it could also be that there are already published best-practice pipelines out there (e.g. GATK4) that perform similar tasks to your desired workflow. So it is worthwhile before starting to read around the area, to see what else has been done.

Do any similar open source workflows exists (even in another language)?

Consult:
* <https://www.myexperiment.org/home>
* <https://dockstore.org/> 
* <https://workflowhub.eu/>
* <https://nf-co.re/>
* ..

## Start with high level organisation

Workflow design should start off with abstract high-level schematics - sketching out rough steps required [using abstract operations as placeholders](https://docs.bioexcel.eu/cwl-best-practice-guide/devpractice/partial.html#using-abstract-operations-as-placeholders) to quick create a valid, if not yet runnable, workflow. 

Once you have your valid high-level workflow, you then can take advantage of CWL features such as [nesting workflows](https://docs.bioexcel.eu/cwl-best-practice-guide/devpractice/partial.html#using-nested-workflows) to fill in the details with further abstract operations, until you have a near complete rough layout of the workflow.

## What will be the inputs and outputs data and formats? 

A major part of sketching out the high-level organisation for the workflow will be identifying what inputs and outputs the workflow will require/produce, and defining what formats and layout these should take.

Obvious questions to answer are, what formats will these inputs and outputs have, can you control these with any entomology definitions? You may also need to consider naming conventions or directory structures, if you will be parallelising your workflow using scattering then you need to consider how you get information into that scatter step, and how you ensure that you do not overwrite the output from different scattered processes with that from the last process to finish. Another major question, especially when working with cloud computing resources, is what are the storage/access requirements for this data? Addressing questions of how you intend to move data to/from your running workflow now will save a lot of stress later.

## What will be (if any) the intermediate data and formats?

CWL steps operate in discrete environments, and any data from one step needed for the next has to be explicitly specified in the workflow (it is the requirement on which step provides inputs for which other steps that CWL uses to determine execution order for your workflow). So in your sketch you should include the intermediate files produced by a step which are needed by the next (this may not be possible at the start if you're writing a workflow for a new tool, but it should be a priority when adding details to new workflows). The same considerations of formatting and entomologies should be made for intermediate data as for the whole workflow inputs and outputs.

Considerations of how intermediate data will be moved between steps in your workflow may also be necessary. On shared filesystems (such as local or HPC facilities) this may be straightforward (although a number of HPC facilities do make use of high-performance on-node storage to increase IO performance - which could complicate workflows if you try to minimise compute costs). For cloud-based services, however, you again should consider how data will be moved between individual steps.

## What are the execution restrictions?

Can it run on the public cloud? Is data confidential? Cost/time limits?
Can workflow be developed in open?

## Making a valid non-executable workflow

Version 1.2 of CWL introduces the [Operation](https://www.commonwl.org/v1.2/Workflow.html#Operation) class. This can be used to represent a potential step in a workflow which does not yet have a CommandLineTool, Workflow or ExpressionTool implementation. It allows for a workflow to be created, which will not be executable, but will be valid for the purposes of running analysis of the workflow, such as printing RDF graphs, or workflow visualization.

The use of this class is described further in [Writing Partial Workflows](https://docs.bioexcel.eu/cwl-best-practice-guide/devpractice/partial.html#using-abstract-operations-as-placeholders).


