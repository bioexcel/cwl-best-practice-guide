---
layout: default
title: Introduction
---

# Introduction

CWL has been designed to be a platform-agnostic tool for building (principally scientific) workflows. Taking tools such as make as its inspiration, but applying these principles to the very heterogeneous infrastructure that is available today. It is also intended to help facilitate the tracking of provenance for these workflows, enabling research to be more reproducable. Both of these aims are achieved by isolating the individual tasks in the workflow - requiring the explicit specification of the inputs and outputs from each step assists with provenance tracking, as well as allowing each task to be run on the most suitable platform.

## Overview of Common Workflow Language

CWL is an abstract language, consisting of a clearly defined set of standards, to which all implementations must conform. At the time of writing, the most recent version of these standards is v1.2, as defined in the [Workflow Description Standard](https://www.commonwl.org/v1.2/Workflow.html) and [Command Line Tool Description Standard](https://www.commonwl.org/v1.2/CommandLineTool.html).

CWL workflows are written in a mix of YAML and JSON, following the [Semantic Annotations for Linked Avro Data (SALAD)](https://www.commonwl.org/v1.2/SchemaSalad.html) schema language. The features of these languages which are used in CWL are summarised in the [user guide](https://www.commonwl.org/user_guide/yaml/); this introduction should be enough to get most people started with writing CWL workflows. 

The design philosophy is for the workflow to be as modular as possible - workflows can consist of a nested collection of CWL scripts, with an overarching script describing the whole workflow calling on other scripts which describe each tool used, or are smaller workflows themselves which refer on to other scripts which describe the tools used.

Each CWL script starts with a definition of the version used, and class for the script, e.g.:
```
cwlVersion: v1.0
class: Workflow
```
The engine used will check for compliance with the specified version of the language. All stable versions of the langauge are backwards compatible with previous versions (care should be taken using development versions though, as features in these may not make it to the final version, or may work differently). Not all engines are yet compliant with the latest language definition - please check the engine guide for more details on this.

Within a workflow different language versions can be specified for each script, allowing for libraries of tools to be shared between projects without requiring adaption for newer projects using the latest version of the language.


## Common CWL use-cases



## Considerations for scalable workflow execution


