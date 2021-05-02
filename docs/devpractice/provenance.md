---
title: Capture provenance from workflow runs
sort: 10
---

# Capture provenance from workflow runs

An important aspect of reproducibility of research is the capturing of information on the tools used, as well as the data, scripts, and configuration settings for these, as well as attribution for who has run the workflow. Doing this manually is time consuming, and difficult to carry out in a systematic manner, so for many systems / workflow engines, tools have been developed to help with the capturing of this retrospective provenance. CWL has also had a provenance tool developed for it, [CWLProv](https://doi.org/10.1093/gigascience/giz095). The intention is that this will provide a cross-platform / engine provenance tool. 

Provenance for a CWL script can be captured using the command:
```
cwl-runner --provenance [prov] workflow.cwl workflow_settings.yml
```
This creates the `[prov]` directory.


## Provenance of test runs

## Provenance of real runs

Handling large data with Git LFS
