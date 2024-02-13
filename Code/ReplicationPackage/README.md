## [Supplementary material] AGORA: Automated Generation of Test Oracles for REST APIs

This is the supplementary material of the paper entitled *AGORA: Automated Generation of Test Oracles for REST APIs*

**NOTE:** We recommend navigating this documentation using VS code (which preserves hyperlinks and renders Markdown).

## Contents

1. [Getting Started](#getting-started)
1. [Detailed Instructions](#detailed-instructions)
    1. [Experiment 1: Test oracle generation](#experiment-1-test-oracle-generation)
    1. [Experiment 2: Failure detection](#experiment-2-failure-detection)
1. [Package structure](#package-structure)
    1. [Bug reports](#bug-reports)
    1. [Data Analysis Python Project](#dataAnalysisPythonProject)
    1. [Datasets](#datasets)
    1. [Documentation](#documentation)
    1. [Java projects](#java-projects)
    1. [Reports](#reports)
    1. [RESTest configuration files](#restest-configuration-files)
    1. [Videos](#videos)

# Getting Started
This section describes how to set up the artifact and validate its general functionality. These experiments can be replicated using the provided source code of the different Java projects (see subsection [Java projects](#java-projects)) or the provided Ubuntu virtual machine (https://doi.org/10.5281/zenodo.7641127).

We provided video tutorials for each one of the projects (Beet, Daikon and JSONMutator, see subsection [Videos](#videos)). Each one of these videos explains how to replicate our experiments with a running example.

# Detailed Instructions
This section describes how to validate our paper's claims and results in detail.

The Java projects provided in the Ubuntu virtual machine and in the [Java projects](#java-projects) folder are the versions used for our experiments. Please refer to https://www.github.com/isa-group/Beet if you are interested in using the most up-to-date version.

## Experiment 1: Test oracle generation
This experiment aims to measure the effectiveness of AGORA in generating test oracles for different target operations, using datasets of different sizes (50, 100, 500, 1K and 10K test cases) to analyze the impact of the dataset size in the precision of the approach. 

Each one of the reported invariants was classified as true positive, false positives, enter (it only reveals information about the input parameter and thus it is not considered for computing the precision) or bug (invariant that reveals inconsistent behavior).

1. **Step 1: Execute Beet.** The first step is to execute Beet for each API operation to generate a declaration and a data trace file that can be used as inputs to our modified version of Daikon. The directory `src/test/resources/evaluationOracles` contains, for every API operation the required files that Beet receives as input (i.e., OpenAPI specification and set of test cases in CSV format). 

    For instance, the directory `src/test/resources/evaluationOracles/Spotify/createPlaylist/50` contains the CSV file with 50 test cases for the createPlaylist operation of the Spotify API. Please refer to the [videos](#videos) for more details on how to run Beet. This execution will generate a declaration and data trace files (already provided).

1. **Step 2: Execute Daikon.** The instrumentation (declaration file + data trace file) returned by Beet is processed by our modified version of Daikon to return the list of invariants reported for each API operation in CSV format. For each API operation, provide the absolute path to the declaration file and the data trace file, as done in the provided examples. Set the parameter `use_modified_daikon_version` to true to use our version and to false to use the default one (that we used as a baseline). 

    The reported invariants CSV files are then classified as true positives, false positives, enter or bug. The `reports/invariantsClassified` directory of this package contains these datasets classified.

    Please refer to the [videos](#videos) for more details on how to run Daikon. The version of Daikon provided in our Ubuntu VM already contains the paths of every declaration and data trace files. The [official Daikon user manual](https://plse.cs.washington.edu/daikon/download/doc/daikon/) explains how to install Daikon in your device. However, we recommend using our Docker image to perform this step, (please refer to the Document **DaikonImage.md**).

    In some exceptional cases, invariants related to empty arrays may be reported twice (Daikon reports the invariant for both the pointer and values variable of the array). For example, in the createPlaylist operation of the Spotify API, Daikon reported the invariant `return.followers.total == size(return.images[])` twice. These duplicates were removing when labelling the dataset.

1. **Step 3: Graphic reports.** The `agora-dashboard` Python project contains the source code used for generating the reports and graphs provided in the paper. More details in the [Data Analysis Python project](#dataanalysispythonproject) section.

## Experiment 2: Failure detection
This experiments uses JSONMutator to determine the effectiveness of the test oracles generated by AGORA in detecting failures. Please refer to the [JSONMutator](#videos) video for a step-by-step guide on how to replicate this experiment.

JSONMutator source code is available in the [Java projects](#java-projects) directory and in the Ubuntu Virtual Machine, with an example of 10 repetitions. The `reportsExperiment2_100Executions.zip` file inside the [reports](#reports) directory contains the data resulting from the 100 executions of this experiment (the one used in the paper), along with the average Failure Detection Ratio per API operation.

# Package structure
This sections describes the content of the different directories of this replication package.

## Bug reports

This directory contains anonymized reports of the real faults detected in 5 commercial APIs (Amadeus Hotel, GitHub, OMDb, Marvel and YouTube),
as well as videos that show how to replicate these bugs.

## dataAnalysisPythonProject
This directory contains the Python project used to perform some of our data analysis tasks, such as computing the precision of each API operation with each API configuration, the sunburst in the paper, and the evolution of the precision of the approach. 

- Execute the `compute_precision.py` file to generate a CSV file containing the results of Experiment 1 (Table 2 in our paper). Change the value of the `configuration` variable in this file to `modified` or `original` to generate the precision reports of the modified or the default version of Daikon, respectively.

- Execute the `main.py` file and open localhost:8050 to visualize a configurable dashboard (it is possible to visualize the different graphs filtering by API operation and number of test cases):

![Dashboard](https://imgur.com/xsl4YP9.png)

- Execute the `precision_evolution.py` file to generate the graph that shows the evolution of the precision of AGORA (Figure 3 in our paper).

These reports are generated by analyzing the `labelled_data` directory of the project, that contains the labelled datasets with the invariants classified as TP, FP, enter or bug. Tthe content of this directory is identical to the one provided in the [Reports](#reports) folder.


## Datasets

This directory contains the datasets used for our experiments, namely the different test suites used for experiment 1 (divided in
sets of 50, 100, 500, 1000 and 10K requests), the test suites that were mutated for experiment 2, and our invariant taxonomy.

## Documentation

This directory contains the following documentation:

* **Beet wiki:** Extended running example showcasing the functionality of the Beet instrumenter. The latest version of this wiki can be found [here](https://github.com/isa-group/Beet/wiki).
* **JSONMutator configuration:** Description of the mutation operators used for our evaluation (experiment 2).
* **Modifications on Daikon:** Detailed description of the modifications performed on the Daikon software tool.

## Java projects

Contains the Java projects that compose our approach (Beet and Daikon modified) and JSONMutator. These projects contain all the files necessary to
replicate our experiments (see "Videos" directory for more details). These projects require Java 8. Consult the official Daikon user manual to obtain more
details about how to configure Daikon. However, we recommend using the Daikon Docker image (please refer to the Document **DaikonImage.md**).

In order to make the replication of our experiments easier, we provide a **Ubuntu Virtual Machine** with all the projects configured. This virtual machine can
be found at: https://doi.org/10.5281/zenodo.7641127 (user: agora, password: agora).

## Reports

This directory contains the reports resulting from our experiments.

* **Invariants classified:** Contains the invariants reported by AGORA and the default version of Daikon in all the API operations. These invariants have been classified as true positives and false positives.
* **Reports experiment 2 100 execution:** Contains the mutation reports generated after the execution of experiment 2.

## RESTest configuration files

This directory contains the OAS specifications, data dictionaries and test configuration files used for generating the test cases used for our experiments. Please refer to the [RESTest documentation](https://github.com/isa-group/RESTest/wiki) for details on how to generate test cases with these configuration files.

## Videos

This directory contains tutorials that show how to use the developed software (Beet, Daikon modified and JSONMutator) and how to replicate our experiments.

These videos were recorded in the provided Virtual Machine.