---
title: Pitfalls and limitations
---

# Common pitfalls and limitations of CWL

CWL works on abstract objects - it can sometimes be unclear how you can pull simple information, such as a file name or path, out from an object to use in another part of the workflow.

## Common CWL workarounds

Javascript ExpressionTools can be used for working around some limitations. It is recommended to keep usage of these to a minimum, but sometimes they are the best tool for working around a problem.

## When CWL might not be best workflow language

CWL is a data-driven 'dataflow' workflow standard - dependence between steps, and how they run, is controlled by the passing of data. As such the language does not have some of the flexibility in operation that a control-flow language (e.g. looping through steps repeatedly, or cancelling steps due to certain criteria). The [Workflow Pattern Initiative](http://www.workflowpatterns.com/) documents many of the different workflows which are possible. CWL does not cover all types of workflow pattern, but an analysis of how [CWL match the Control and Data patterns](https://github.com/common-workflow-library/cwl-patterns/tree/master/workflow_patterns_initiative) has been carried out, which could be helpful for you in deciding if CWL will provide the control you need for your workflow.

## Dealing with type incompatibility, e.g. mismatch of EDAM format

## Managing file names and file storage


## EDAM format gotchas

Adding metadata enforces type checking  - what workarounds?
