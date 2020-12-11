---
title: Writing partial workflows
sort: 5
---

# Writing partial workflows

To make development an easier process, in which you can test your code throughout development, it is good practice to break the workflow down into discrete steps that can be written and tested individually. CWL enables the use of nested scripts, so that you can do this partial development more easily.

To split out your operations to separate scripts, your master script should follow the design:

```yaml
class: Workflow
inputs:
  val_in: string
  
outputs:
  val_out:
    type: string
    outputSource: step2/out2

steps:

  step1:
    in:
      in1: val_in
    run: subscriptA.cwl
    out: [out1]
    
  step2:
    in:
      in2: step1/out1
    run: subscriptB.cwl
    out: [out2]
```
The individual tasks within the workflow will then be described in scripts `subscriptA.cwl` and `subscriptB.cwl`. These can be tool wrappers, expression tools, or full workflows (with calls to further CWL scripts) themselves.



## Maintaining and reusing tool descriptions

One important use for these subscript workflows is the recording of tool API descriptions, easing the reuse of these in multiple workflows. These scripts will follow the format:
```yaml
class: CommandLineTool
label: example tool wrapper
doc: |
  Description of the tool, and how to use it.
baseCommand: tool_example
hints:
  DockerRequirement:
    dockerPull: docker_container_name_if_needed:latest

inputs:
  input_a:
    label: input description
    doc: |
      Description of input_a, including type, etc.
    type: File
    inputBinding:
      position: 2
  
  input_b:
    label: input description
    doc: |
      Description of input_b, including type, etc.
    type: string
    inputBinding:
      position: 1
      prefix: --config_string

outputs:
  output_a:
    label: output description
    doc: |
      Description of output_a
    type: File
  outputBinding:
    glob: "examplefile.bin"

```

Examples of such tool descriptors can be found in the BioExcel Building Block adapters repository: <https://github.com/bioexcel/biobb_adapters/tree/master/biobb_adapters/cwl>.

These tool descriptions can then be called from an overarching workflow script (assuming the above script has been saved as `exampletoolwrapper.cwl`):
```yaml
class: Workflow
label: Example workflow calling a tool descriptor
doc: |
  Description of example script.
  
inputs:
  step1_file: file
  step1_config: string

outputs:
  binary_out:
    label: example output file
    doc: |
      description of output file (can be copied from tool description script)
    type: File
    outputSource: step1_example/output_a

steps:
  step1_example:
    label: running example tool
    doc: |
      description of tool (can be copied from tool description script)
    run: path/to/library/exampletoolwrapper.cwl
    in:
      input_a: step1_file
      input_b: step1_config
    out: [output_a]
```

Using this method of wrapping tools enables you to keep details such as docker container requirements separate from your main workflow script, making the overarching script as clean as possible. It also enables you, if you are running your scripts on a host with internet access, to use tool descriptions stored remotely. E.g.:
```yaml
    run: biobb_adaptors/cwl/biobb_io/mmb_api/pdb.cwl
```
was be replaced with:
```yaml
    run: https://raw.githubusercontent.com/bioexcel/biobb_adapters/v0.1.4/biobb_adapters/cwl/biobb_io/mmb_api/pdb.cwl
```
In this tutorial BioBB CWL script: <https://github.com/bioexcel/biobb-cwl-tutorial-template/blob/master/BioExcel-CWL-firstWorkflow.cwl>. Doing this reduces library installation requirements for your endusers.



## Using nested workflows

More complex workflows can be constructed by nesting another workflow within the called CWL script. These should be laid out in a similar manner to a standard workflow. The difference in use will be that the inputs are read from the parent workflow, and the outputs are passed to that same workflow, rather than directly to the user.

For example, consider that `subscriptA.cwl` from the example above is a workflow too. The script could be laid out as:
```yaml
class: Workflow
inputs:
  in1: string
  
outputs:
  out1:
    type: string
    outputSource: step3/outstring3

steps:

  step1:
    in:
      inputa: in1
    run: subsubscriptA.cwl
    out: [outstring1]
    
  step2:
    in:
      inputb: step1/outstring1
    run: subsubscriptB.cwl
    out: [outstring2]
    
  step3:
    in:
      inputc1: step1/outstring1
      inputc2: step2/outstring2
    run: subsubscriptC.cwl
    out: [outstring3]
```
In this case the scripts `subsubscriptA.cwl`, `subsubscriptB.cwl`, and `subsubscriptC.cwl` could all again be either tool wrappers, expression tools, or full workflows again.

## Using abstract operations as placeholders

Version 1.2 of CWL introduces the [Operation](https://www.commonwl.org/v1.2/Workflow.html#Operation) class. This can be used to represent a potential step in a workflow which does not yet have a CommandLineTool, Workflow or ExpressionTool implementation. It allows for a workflow to be created, which will not be executable, but will be valid for the purposes of running analysis of the workflow, such as printing RDF graphs, or workflow visualization.

If you were developing a workflow which used the `exampletoolwrapper.cwl` script given above, you might start developing that tool wrapper by first sketching out the inputs and outputs that you want, using the `Operation` class:
```yaml
cwlVersion: v1.2
class: Operation
label: example tool wrapper
doc: |
  Description of the tool, and how to use it.

inputs:
  input_a:
    label: input description
    doc: |
      Description of input_a, including type, etc.
    type: File
  
  input_b:
    label: input description
    doc: |
      Description of input_b, including type, etc.
    type: string

outputs:
  output_a:
    label: output description
    doc: |
      Description of output_a
    type: File
  outputBinding:
    glob: "examplefile.bin"
```
Here we have included the input and outputs, with labels and documentation so that we know what these should be. Not included are the `baseCommand` (as this isn't in the `Operation` standard), nor the hints (which could be included, but perhaps we are not yet aware of the docker requirement), nor the `inputBinding`s for each input (as we haven't yet seen a working example of the tool we are wrapping). We have also specified the cwlVersion should be v1.2, to make sure our cwl-runner implementation is prepared for the `Operation` class.

Including `Operation` scripts like this will allow the workflow script that calls it to be run using cwl-runner - however, as `exampletoolwrapper.cwl` is abstract, all you would get for a 'successful' run is the warning message `Workflow has unrunnable abstract Operation`.

However, as the workflow is now valid, you can run analysis tools on it, such as validating the workflow, printing sub-graphs, checking input requirements, etc. So starting off with `Operation` class scripts within your workflow enables higher level development to take place.

## Workflow/data variants using top-level workflows

Remembering that CWL scripts can be nested helps when adapting existing workflows to new data or uses. The most obvious use case here is for working through a range of scenarios using [scatter to parallelise your workflow](https://docs.bioexcel.eu/cwl-best-practice-guide/devpractice/scalability.html#where-to-scatter-structuring-iterations-with-nested-workflows). However you should remember that you can use the same principles even when you are adapting a workflow for a single change in input, where it might be clearer to add an extra preprocessing step to a top-level workflow that calls an existing workflow, rather than modifying the original workflow for this single use case. 
