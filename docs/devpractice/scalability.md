---
title: Ensuring workflow is scalable
sort: 8
---

# Ensuring workflow is scalable

When you have developed your workflow to the point of giving
promising results, verified by [running](running.md) with 
your [test data](tests.md) it's time to scale up. 

One of the advantages of using a workflow management system
with [reproducibility](running.md) 
and [interoperability](interoperability.md) features is that a workflow running
locally can be transferred to run on a larger compute infrastructure,
without requiring significant changes.  

This enables further development of the same
workflow stay in sync across deployments.  A workflow
where all 
[dataflows are explicit](interoperability.md#avoiding-off-workflow-data-flows) 
and [tools](tools.md) run in independent containers 
also allows job-level "embarrassingly parallel" upscaling 
without needing to handle job queues or file moving.

However, simply executing a workflow at a larger compute
infrastructure will not always give your
workflow an automatic boost; the way you structure
your workflow and annotate it with execution hint will help
the engine along with utillizing your compute resources more 
efficiently.

## Choosing compute infrastructure and workflow engine

The optimizations appropriate for your workflow will partially
depend on  which particular compute infrastructure, storage solution
and workflow engine you have settled on.


While the reference engine `cwltool` aims to support all of CWL's features, 
it is by design a single-node local executor that focus on correctness and workflow 
debugging. By default `cwltool` run one tool invocation at a time, and "fail fast"
the whole workflow on any error. This can be tweaked with 
`--parallel --on-error continue` which, together with
`--cachedir` on repeated invocations, may provide significant speedups when 
testing a workflow locally. Nevertheless, for larger computational 
workflows it will be necessary to execute across multiple nodes 
on a local cluster or cloud infrastructure.

For the Common Workflow Language,
[multiple implementations](https://www.commonwl.org/#Implementations) are
available, with support for many different computational backends. 
These engines vary in their complexity, features and documentation
available for different scenarios and there is not a particular engine
that is always the best choice. 

The BioExcel Best Practice Guide 
[How to choose which CWL engine to deploy](https://docs.bioexcel.eu/cwl-engine-guide/)
covers the main choices of workflow engines for running CWL workflows, 
but you may also want to do your own research and trials 
depending on your particular setup and requirement.

If you have followed the [interoperability](interoperability.md) advice then
your CWL workflow should work in all compliant engines, but that does not
mean you should compare their performance based only on a workflow
that worked sufficently on a desktop computer.

The rest of this page show additional hints and techniques that 
can be used to help your workflow engine in efficiently 
executing your workflow.

## Handle large-scale scattering



## Considerations for data handling on cloud execution

<https://doc.arvados.org/user/cwl/cwl-extensions.html>


## Where to scatter? Structuring iterations with nested workflows

The principal best followed for scattering workflows is to try to create the longest continuous workflows possible. Scatter instructions should enclose workflows where each step relies only on global outputs from steps previous to the scatter instruction, or on local outputs from steps after the scatter instruction. Creating a workflow where each step is scattered individually is inefficient, as each step will take as long as the longest individual process.

Scattering can be best organised using nested workflows - creating a single workflow which covers all tasks within a scattered process, then scattering that will keep the dependencies clear.

An example external script:
```yaml
cwlVersion: v1.0
class: Workflow

requirements:
  SubworkflowFeatureRequirement: {}
  ScatterFeatureRequirement: {}

inputs:
  step_input_files:
    type:
      type: array
      items: File
  input_config: string

outputs:
  output_file:
    type: File
    outputSource: step_outer/return_file

steps:
  step_outer:
    run: inner_script.cwl
    scatter: input_file
    in:
      input_file: step_input_files
      config_information: input_config
    out: [return_file]
```

And the example internal script:
```yaml
cwlVersion: v1.0
class: Workflow

inputs:
  input_file: File
  config_information: string

outputs:
  return_file:
    type: File
    outputSource: step_two/final_file

steps:
  step_one:
    run: process_one.cwl
    in:
      file_prepare: input_file
      prep_config: config_information
    out: [intermediate_file]

  step_two:
    run: process_two.cwl
    in:
      file_process: step_one/intermediate_file
    out: [final_file]

```
In this example, because step_two depends only on the single intermediate_file generated by step_one (and not on the intermediate_file generated by any other of the scattered workflows), we enclose these two steps within the same scatter step.

## Handling many files

