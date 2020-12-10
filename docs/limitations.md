---
title: Pitfalls and limitations
---

# Common pitfalls and limitations of CWL

CWL works on abstract data objects - it can sometimes be unclear how you can pull simple information, such as a file name or path, out from an object to use in another part of the workflow.

CWL will not, except when using ontologies such as the EDAM format, check data object contents. To do this you would have to add extra tasks into your workflow, to check the outputs from a previous task to ensure they are suitable for the next task. These checks could never trigger a re-execution of the previous task though, all that could be done in the case of an error is to raise a permanent failure of the whole workflow process.

In CWL each task is carried out within it's own environment. Any data inputs or outputs (I/O) from that task must be explicitly defined. There is no concept of scopes or cases within which tasks might operate, and so there is no way to have [scope data](http://www.workflowpatterns.com/patterns/data/visibility/wdp3.php) or [case data](http://www.workflowpatterns.com/patterns/data/visibility/wdp5.php) which is available to all tasks within that environment. This requirement for explicit I/O declarations helps with tracking provenance of tasks, but can feel very awkard and require what seems to be excess verbosity in the workflow script if you are used to setting up a running environment once, then carrying out all tasks within that environment.

## Common CWL workarounds

Javascript [ExpressionTools](https://www.commonwl.org/v1.2/Workflow.html#Expressions_(Optional)) can be used for working around some limitations of CWL. It is recommended to keep usage of these to a minimum, as their implementation in the standard is optional and so using them may limit the portability of your workflow, but sometimes they are the best tool for a task. If you do need to use them there is guidance on doing this in the [CWL user guide](https://www.commonwl.org/user_guide/13-expressions/index.html).

## When CWL might not be best workflow language

CWL is a data-driven 'dataflow' workflow standard - dependence between steps, and how they run, is controlled by the passing of data. As such the language does not have some of the flexibility in operation that a control-flow language. The [Workflow Pattern Initiative (WPI)](http://www.workflowpatterns.com/) documents many of the different workflows which are possible. A review of how [CWL match the Control and Data patterns](https://github.com/common-workflow-library/cwl-patterns/tree/master/workflow_patterns_initiative) described in the WPI has been carried out; this is a good detailed guide to how CWL functionality fits into the landscape of possible workflows. 

When designing CWL workflows it is important to keep at the front of your mind that the flow of tasks is entirely controlled using the data objects within that flow. Tasks start when they have all required input data objects, and when they end the workflow will continue to the next task if all required data objects have been returned. CWL does not determine task execution using any other criteria than these. Nor does CWL allow for the exchange of data objects between tasks once execution has started - all required data objects must be present when a task starts, and no data objects, optional or otherwise, are sent or received by a task during task execution.

This means that many control workflow patterns are unusable. Obvious examples of these limitations are the lack of [loops](http://www.workflowpatterns.com/patterns/control/new/wcp21.php) and [recursive](http://www.workflowpatterns.com/patterns/control/new/wcp22.php) patterns; to replicate such structures in CWL you would need a priori knowledge of how many iterations of the loop or recursion were required, and then create that many duplicated tasks, each passing a copy of the same data objects, uniquely labelled for each iteration of the process. In addition, because [cancelling tasks](http://www.workflowpatterns.com/patterns/control/cancellation/wcp19.php) according to given criteria is not possible, usecases such as [structured discriminators](http://www.workflowpatterns.com/patterns/control/advanced_branching/wcp9.php) where diverged branches of the same workflow are reconverged once a certain criteria is reached; [deferred choice](http://www.workflowpatterns.com/patterns/control/state/wcp16.php) where the choice on which branch to follow is made simply on which starts first, and all other branches are cancelled after the fact; or even [explicit termination](http://www.workflowpatterns.com/patterns/control/new/wcp43.php) of a workflow, are also not possible.

The [conditional execution](https://www.commonwl.org/v1.2/Workflow.html#Conditional_execution_(Optional)) feature has been added to the WorkflowStep definition in version 1.2 of the CWL standard. This adds a `when` field to the workflow, which will contain an expression evaluated using the inputs to that task, and returns a boolean value which determines if the task will be run or not. The addition of this feature allows for workflow patterns such as [exclusive choice](http://www.workflowpatterns.com/patterns/control/basic/wcp4.php) or even [multi-choice](http://www.workflowpatterns.com/patterns/control/advanced_branching/wcp6.php) branching, based on the data objects provided by previous tasks, to be used. However, this feature is an optional part of the language, and is very new, so using it in your workflows could severly limit the portability of the workflow.

## Dealing with type incompatibility, e.g. mismatch of EDAM format

## Managing file names and file storage


## EDAM format gotchas

Adding metadata enforces type checking  - what workarounds?
