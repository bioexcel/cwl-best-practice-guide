---
title: Running and verifying workflow
sort: 6
---

# Running and verifying workflow

## Using job files

## Inspecting intermediate results

Intermediate data for a workflow can be generated by calling the CWL reference implementation `cwl-runner` with the `--cachedir` option. This will cache all step outputs, and will re-use these cached outputs where possible, so that if you have to restart a workflow then this computational work will not have to be repeated.

## Using CWLProv


