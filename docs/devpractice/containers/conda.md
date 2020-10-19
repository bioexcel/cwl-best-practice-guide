---
title: Conda and BioConda
sort: 2
---

# Conda & BioConda

An alternative to [containers](docker.md) 
is to use a packaging system like 
[Conda](https://conda.io/), particularly as the 
[BioConda](https://bioconda.github.io/) initiative have wrapped an 
more than 7000 bioinformatics packages and are frequently updated.

Compared to containers a Conda environment is also a file hierarchy 
with its own `bin` and `lib`, but activating an environment is less 
intrusive and requires no `root` permissions as it only modifies
shell environments like `PATH`. Conda environments do not gain the
safety from isolation as in containers, but typically do include most
of the system dependencies like `libstdc.so`.

## Conda in CWL

To indicate in CWL that a package is available from Conda, we add a 
[SoftwareRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#SoftwareRequirement)
item to the `hints` of the `CommandLineTool` object.


```tip
We could place the `SoftwareRequirement` under `requirements` instead, but 
this would prevent workflow execution if Conda was not available, even if the
command line was available on `PATH`. Similarly we also usually
place `DockerRequirement` under `hints` so that the workflow engine can
try both options.
```

In CWL, Conda dependencies are identified by their URL in a
https://anaconda.org/ _channel_ - which typically should be one of:

* <https://anaconda.org/conda-forge>
* <https://anaconda.org/bioconda/>


```yaml
hints:
  SoftwareRequirement:
    packages:
      curl:
        specs: 
          - https://anaconda.org/conda-forge/curl
```

## Using Conda in the shell

One advantage of Conda is that it can also be easily used on
the command line tool for experimenting with the same version
of the tool as in the workflow:

```shell
(base) stain@biggie:~$ conda create -n bam bamtools
 (..)
(base) stain@biggie:~$ conda activate bam

(bam) stain@biggie:~$ type bamtools 
bamtools is hashed (/home/stain/miniconda3/envs/bam/bin/bamtools)

(bam) stain@biggie:~$ bamtools  --version
bamtools 2.5.1
```

The binaries will mainly reference the Conda environment `/home/stain/miniconda3/envs/bam/`
but will also refecence some core libraries in the main operating system.

```shell
(bam) stain@biggie:~$ ldd /home/stain/miniconda3/envs/bam/bin/bamtools
	linux-vdso.so.1 (0x00007ffdcc54d000)
	libz.so.1 => /home/stain/miniconda3/envs/bam/bin/../lib/libz.so.1 (0x00007f5c13290000)
	libstdc++.so.6 => /home/stain/miniconda3/envs/bam/bin/../lib/libstdc++.so.6 (0x00007f5c1311b000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f5c12fb4000)
	libgcc_s.so.1 => /home/stain/miniconda3/envs/bam/bin/../lib/libgcc_s.so.1 (0x00007f5c12fa0000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5c12dae000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f5c133fe000)
```

## Conda tips and caveats

```tip
If a [BioConda](https://bioconda.github.io/) package is outdated or missing,
[contributing a recipe](https://bioconda.github.io/contributor/index.html) is often
welcomed in a couple of hours by a helpful community.
```
One disadvantage of Conda is that it may take long to download and initialize
from an empty environment. It is also slightly more fragile than containers, as
you can install multiple tools, or later updates
could cause library dependency version conflicts. CWL engines using Conda will 
create a new environment for each CWL `CommandLineTool`.

It is possible to list multiple conda `packages` in CWL, although
usually the main recipe will have all the required dependencies included.

Conda can install the CWL engines [conda-forge/cwltool](https://anaconda.org/conda-forge/cwltool)
and [bioconda/toil](https://anaconda.org/bioconda/toil), and they 
can be used to run CWL workflows that use Conda or Docker.

```warning
As `bioconda/toil` depend on a particular version of `cwltool`, install that first. If you desire
a newer `cwltool` create a separate Conda environment. Install 
[conda-forge/cwltool](https://anaconda.org/conda-forge/cwltool), 
not the outdated [bioconda/cwltool](https://anaconda.org/bioconda/cwltool)!
```

Conda packages are compiled for each operating system and may have subtle differences. 
BioConda packages are only available for macOS and Linux. 

Conda can be used to get a consistent/updated set of 
GNU [coreutils](https://anaconda.org/conda-forge/coreutils),
[sed](https://anaconda.org/conda-forge/sed), [Python](https://anaconda.org/conda-forge/python) 
etc. on macOS and Windows.


