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


## Maintaining and reusing tool descriptions

One important use for these subscript workflows is the recording of tool API descriptions, easing the reuse of these in multiple workflows. These scripts will follow the format:
```
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
```
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
```
    run: biobb_adaptors/cwl/biobb_io/mmb_api/pdb.cwl
```
was be replaced with:
```
    run: https://raw.githubusercontent.com/bioexcel/biobb_adapters/v0.1.4/biobb_adapters/cwl/biobb_io/mmb_api/pdb.cwl
```
In this tutorial BioBB CWL script: <https://github.com/bioexcel/biobb-cwl-tutorial-template/blob/master/BioExcel-CWL-firstWorkflow.cwl>. Doing this reduces library installation requirements for your endusers.



## Using nested workflows

## Using abstract operations as placeholders

## Workflow/data variants using top-level workflows

