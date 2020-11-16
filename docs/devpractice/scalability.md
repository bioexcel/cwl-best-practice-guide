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

The BioExcel Best Pratice Guide 
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

<https://doc.arvados.org/v2.1/user/cwl/cwl-extensions.html>


## Where to scatter? Structuring iterations with nested workflows

## Handling many files

