---
title: Writing partial workflows
sort: 5
---

# Writing partial workflows

To make development an easier process, in which you can test your code throughout development, it is good practice to break the workflow down into descrete steps that can be written and tested individually. CWL enables the use of nested scripts, so that you can do this partial development more easily.

To split out your operations to separate scripts, your master script should follow the design:

```
class: Workflow
inputs:
  val_in: string
  
outputs:
  val_out:
    type: string
    outputSource: step1/out1

steps:

  step1:
    in:
      in1: val_in
    run: subscript.cwl
    out: [out1]
```


## Maintaining and reusing tool descriptions



## Using nested workflows

## Using abstract operations as placeholders

## Workflow/data variants using top-level workflows

