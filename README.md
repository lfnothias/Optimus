#Optimus: a workflow for LC-MS feature analysis and spatial mapping

<img src="img/workflow.png"/>

###Table of contents

* [Introduction](#introduction)
* [Who might need this workflow?](#who-needs-this-workflow)
* [What it does?](#what-it-does)
* [What it doesn't do? (so far)](#what-it-doesnt-do-so-far)
* [System requirements](#system-requirements)
* [Installation](#installation)
  * [Express installation (Windows and OS X >= 10.10)](#express-installation--windows-7-or--os-x-1010)
  * [Regular installation](#regular-installation)
  * [Installing and updating workflow](#installing-and-updating-workflow)
* [Input](#input)
* [IMPORTANT: Stub input file](#important-stub-input-file)
* [Basic use-case](#basic-use-case)
* [Output](#output)
* [KNIME Basics](#knime-basics)
* [Demo](#demo)
* [Advanced use-cases](#advanced-use-cases)
* [Known issues](#known-issues)
* [License](#license)

## Introduction

Optimus is a workflow for LC-MS-based untargeted metabolomics. It can be used for feature detection, quantification, filtering (e.g. removing background features), mz-RT matching with a list of known ions, normalization and, finally, for spatial mapping of detected molecular features in 2D and 3D using the [`ili app](https://github.com/ili-toolbox/ili). Optimus employes the state-of-the-art LC-MS feature detection and quantification algorithms by [OpenMS](http://www.openms.de) which are joined into a handy pipeline with a modern workflow management software [KNIME](https://www.knime.org) with additional features implemented by us.

The workflow is being developed by [Alexandrov Team](http://www.embl.de/research/units/scb/alexandrov/index.html) at EMBL Heidelberg ([contact information](http://www.embl.de/research/units/scb/alexandrov/contact/index.html)).

* Developer: Ivan Protsyuk
* Principal investigator: Theodore Alexandrov

## Who needs this workflow?

The workflow is developed for spatial mapping but can be useful in almost any study of LC-MS-based untargeted metabolomics. It is developed to be open-source, sharable, and efficient enough to process hundreds of LC-MS runs in reasonable time.

## What it does?

The workflow consists of the following consequtive steps:

1. Detection of LC-MS features in each run.
2. Alignment and quantification of features detected across all the runs.
3. (*Optional*) Visualization of the timeline of internal standards intensities.
4. (*Optional*) Exclusion of features that came from blank/control runs.
5. (*Optional*) Exclusion of rarely observed features, i.e. features that occur in a small number of runs only.
6. (*Optional*) Exclusion of features for which MS/MS spectra were not acquired.
7. (*Optional*) Exclusion of features that are not reproducible in pooled quality control (QC) runs.
8. (*Optional*) Imputation of intensities of missing features.
9. (*Optional*) Putative molecular annotation of detected features by mz-RT matching to a list of molecules of interest. This implements a molecular identification at the level *putatively annotated compounds*, corresponding to the level 2 of the Metabolomics Standards Initiative; see [Sumner et al. (2007) Metabolomics, 3(3), 211-221](http://link.springer.com/article/10.1007%2Fs11306-007-0082-2) for details. Note that MS/MS validation of putative annotations is needed (currently not provided in Optimus). The list of molecules of interest can be directly exported from [GNPS](http://gnps.ucsd.edu/) as a result of MS/MS matching against spectral libraries available at GNPS. Alternatively, the list can be provided as a CSV file created manually.
10. (*Optional*) Normalization of intensities of detected features. Currently, several normalization methods are available, based on:
  * total ion current (TIC) of each run;
  * internal standards present in all runs;
  * features detected in pooled QC samples.
11. Creating a heat map of intensities of detected features across all samples.
12. Visualizing samples on a 3D PCA plot.
13. Creating spatial maps of detected features that can be visualized in [`ili app](https://github.com/ili-toolbox/ili). It is a web-application for interactive visualization of spatial data mapped either on an image or a 3D model, also developed by Alexandrov Team.

## What it doesn't do? (so far)

The workflow doesn't support MS/MS-based metabolite identification and adducts deconvolution. We are working on it.

## System requirements

 * *Operating system*: only 64-bit systems are supported; MS Windows, Linux or Apple OS X.
 * *RAM*: 1 GB is minimal amount. Generally, it is not enough for analysis of large datasets containing about a hundred or more LC-MS runs. However, it is sufficient for smaller ones. After all, it very much depends on the data itself (instrument, mass resolution, LC run-time, etc) and workflow settings.
 * *CPU*: no special requirements. The more powerful your CPU is, the faster Optimus works. It leverages effectively multicore processors for parallel execution which improves the overall performance dramatically.
 * *Hard drive*: all Optimus components take about 2.5 GB. During the execution Optimus creates temporary files that can occupy up to few times more space than initial dataset. Those files are not deleted automatically to enable iterative execution and re-execution of Optimus. However, there is an option inside the workflow to clean up temporary files.

## Installation

The workflow is performed by [KNIME Analytics Platform](https://www.knime.org/), an open-source cross-platform general-purpose workflow management system. Before you start using the workflow, you need to install **KNIME** itself, **Python 2.7** (if it's not already installed) and a few additional modules for Python and KNIME. The installation steps are described below. If your computer is running Windows 7 or newer or OS X 10.10 or newer, you can avail express installation scripts described in the section below. Otherwise, you will need to install Optimus dependencies manually as per [Regular installation](#regular-installation) section.

### Express installation (>= Windows 7 or >= OS X 10.10)

**Windows users**: download these two files to the same directory: [win_installer.cmd](https://raw.githubusercontent.com/alexandrovteam/Optimus/master/installer/win_installer.cmd) and [win_core.ps1](https://raw.githubusercontent.com/alexandrovteam/Optimus/master/installer/win_core.ps1). Double-click `win_installer.cmd`, and it should install KNIME and Python automatically. During the installation, you will be prompted to select KNIME installation directory via a graphical window.

**OS X Users:** download [mac_installer.sh](https://raw.githubusercontent.com/alexandrovteam/Optimus/master/installer/mac_installer.sh) on your computer and execute it in terminal with `sudo bash mac_installer.sh`.

**All**: After the installation has finished, and you can proceed to [Installing and updating workflow](#installing-and-updating-workflow) section.

### Regular installation

1. Download and install **Python 2.7 64-bit** if you don't have installed (it can be the case on Windows). You can download it from [the official Python Downloads Page](https://www.python.org/downloads/).
  * You can check easily if the needed Python distribution is already installed by typing `python -V` in your command prompt. You expect to see the output line starting with "Python 2.7". The second part of the version check is determining a bit version of your python interpreter. Follow [this instruction](https://intelligea.wordpress.com/2015/08/05/check-if-python-version-is-64-or-32-bit/) to know whether you have a 64-bit Python or not.
  * *Windows users*: Python installation directory might not be included to your `Path` environment variable. That's why you might get an error message upon executing `python` in command prompt although it's installed. To fix this, you should add <`Python_installation_directory`> to `Path` as well as <`Python_installation_directory\Scripts`>. By default, these directories are `C:\Python27` and `C:\Python27\Scripts`. You can find an instruction on changing `Path` variable [here](http://superuser.com/questions/143119/how-to-add-python-to-the-windows-path).
  * If you know you have a Python distribution installed, but find steps above too complicated, just install another Python interpreter of the needed version. Multiple versions can co-exist on the same computer fine. However, remember the directory where you install it.
2. Install a few Python modules needed for interaction between KNIME and Python. You can do this by executing the command below in your command prompt.
 * *Windows users*: Before, you should download and install a [Microsoft C++ Complier](http://aka.ms/vcpython27). One of those Python modules depends on it.
   * `pip install lxml six pandas protobuf`
 * *Others*: Before, make sure you have `pip` package manager available on your workstation. If you don't, execute `sudo easy_install pip` in the terminal to install it.
    * `sudo pip install lxml six pandas protobuf`
3. Install `pyopenms` Python module which is used by Optimus internally for the retrieval of spectral information from input samples. In order to do this, visit official [pyopenms downloads page](https://pypi.python.org/pypi/pyopenms), scroll down to the list of available versions and download a package that corresponds to your operating system. Afterwards, execute the following command in command prompt to install `pyopenms`: `pip install <path_to_downloaded_package>`
4. Download and install **KNIME Analytics Platform 64-bit**. Select a package according to your operating system on [the official KNIME Downloads Page](https://www.knime.org/downloads/overview?quicktabs_knimed=1#quicktabs-knimed).
 * *Note*: If you already have KNIME installed, make sure that its version is not older than 2.12.*. The workflow hasn't been tested with older versions.
5. Launch KNIME and install additional extensions.
  1. Go to `File => Install KNIME Extensions...`. `Available software` dialog should open after this.
  2. Check the following items in the list of available extensions:
    * `OpenMS`
    * `KNIME Python Integration`
    * `KNIME Quick Forms (legacy)`
    * `KNIME Virtual Nodes`
    * `KNIME JavaScript Views`
  3. Proceed with the installation. You'll be asked to accept the terms of GPL license at the end.
  4. Restart KNIME after the installation is finished.

Note, that the procedure described above should be completed only **once**. So, if you get a new version of the workflow in the future, all you'll have to do is just open it with KNIME. As soon as the steps above are accomplished, your environment is ready to run the workflow.

*Possible Python compatibility issues:* If you have several Python installations in your system, please make sure that KNIME detected the correct one. To do this, go to `File => Preferences`, then type "python" in the filter box. You should see two items at the left-hand side of the dialog: `KNIME > Python`. Click at `Python` and check that there're no error messages appear. If there're any, press `Browse...` and navigate to the python executable that was called when installing modules at the 3rd step. If you followed the instructions above precisely, you can get a path to the needed python executable by executing `which python` in Linux/OS X terminal or `where python` in Windows command prompt.

### Installing and updating workflow

1. Download a zip file "Optimus_v_..." from this repository.
2. In the KNIME window, go to `File => Import KNIME Workflow...`. `Workflow Import Selection` dialog should open after this.
3. Check `Select archive file`, press `Browse...` and select the file downloaded at the 1st step.
  * *Note*: some web-browsers, like Safari, unzip archives automatically after downloading. In this case you'll find a new folder in your "Downloads" folder instead of a zip file. In this case check `Select root directory` in the import dialog, press `Browse...` and select the directory downloaded at the 1st step.
4. Press `Finish`.

Now you should see the `Optimus_v_...` item in the list at the left-hand side of the KNIME window. Double-click on it to open the workflow in the Workflow Editor where you can change its settings and specify input/output files.

For now, new versions of the workflow are uploaded to this repository replacing the previous one. In order to update the workflow on your local computer to a newer version, repeat the steps above.

## Input

The workflow supports the following formats of mass spectrometry data: mzML, mzXML and mzData. Internally, all input files will be converted to mzML, so you can save some time on the workflow execution if your data are already in this format.
Make sure that input files contain centroided data.

### IMPORTANT: Stub input file

The policy of KNIME input nodes implies they always have some files selected. However, it doesn't always match use-cases Optimus can handle, e.g. you might not have blank or pooled QC runs. To bypass this restriction and make the workflow run, you need to create a file called "stub.txt" anywhere on your computer and use it as an input file whenever you don't have files required by an input node. 

## Basic use-case

1. Open the workflow in the KNIME Workflow Editor.
2. Do a right click on the `Read normal samples` node and select `Configure...`. A dialog for input file selection should show up.
3. Press `Clear`, then press `Add` and select files with your samples and press `OK`.
4. Create a [stub file](#user-content-important-stub-input-file).
5. Select the stub file as input for nodes `Read quality control samples`, `Read blank samples`, `Read list of internal standards` and `Read file with mz-RT list`.
6. Click on the `Display feature heat map` node in the Workflow Editor and press `Execute selected and executable nodes` on the KNIME main toolbar. The workflow should start execution after this.
7. After it's finished, right-click on the `Display feature heat map` node and select `Interactive View: Generic JavaScript View`. A window showing distribution of detected features will show up.
  * Note: by default, log transformation is applied to intensity values before rendering them on the heatmap. You can switch to initial intensities using `Scale` at the left-bottom corner of the window.

## Output

In order to save results produced by Optimus, open the configuration dialog of the `Save results` node and specify an output directory. Once the node is executed, it creates 3 files in the output folder. One of them, `features_quantification_matrix.csv` can be opened in any spreadsheet editor (e.g. Excel). Rows in the table will correspond to input samples, whereas columns will represent consensus features, i.e. ions of the same type quantified across the runs. Table cells contain intensities of corresponding features. Names of columns give information on corresponding features. The format is `mz_value RT charge (ID: numeric_identifier)`, so for example a column named `233.112 69 1 (ID: 123)` represents a single-charged ion with mz-value about 233.112 and chromatographic peak at around 69 seconds. As reported features are consensus, it doesn't mean one can find a mass trace in input samples matching consensus mz-value and retention time exactly. These figures are averaged across the runs, but the variation is supposed to be low from run to run.

Numeric identifiers (IDs) are assigned to features after the alignment step and are not changed at the further steps. For the same input dataset and fixed parameters of feature detection and alignment, association between IDs and features are guaranteed to remain the same. So, the IDs can be used as shortcuts for features.

Another file produced by the workflow, `features_isotopic_patterns.csv`, is also a table with the same structure as the quantification matrix. The only difference is table cells contain isotopic patterns of features instead of intensities. The format of an isotopic pattern descriptor is the following: `mz1_min-mz1_max RT1_min-RT1_max|...|mzN_min-mzN_max RTN_min-RTN_max`. In this string, isotopic traces are sorted in ascending order from mz1 to mzN.

The third `*.db` file contains extracted ion chromatograms (XIC) and MS/MS spectra for detected features. The file can be opened with `OptimusViewer` application developed by our team. You can download it and find the instruction on usage in [this GitHub repository](https://github.com/alexandrovteam/OptimusViewer).

## KNIME Basics

If you're new to workflow management systems or KNIME in particular, you can find an introductory tutorial on basic features of KNIME [here](./KNIME Basics.md).

## Demo

This repository contains real-life samples that you can test the workflow on. They're available in this [archive](./examples/3D/apple_samples.zip) (courtesy of Alexey Melnik, Dorrestein Lab, UCSD). Inside, you'll find a directory called `samples` that contains LC-MS samples in mzXML format ready to be processed with the workflow. Blank samples separated from the normal ones in the `blanks` directory inside `samples`. They can be used to remove background features from your result features set.
There're also 2 files in the root folder called `coords.csv` and `Rotten_Apple_Model.stl`. You'll need to supply them at the last step of the workflow that is supposed to produce spatial maps for `ili.

If you want to quickly check, what are actually the results of the workflow, without diving into KNIME and installing everything, you can find the needed file in the `results` folder in the archive. It contains file `features_mapping.csv` which is a spreadsheet containing a table with intensities of different features detected in different runs. This file can be visualized in &#96;ili along with `Rotten_Apple_Model.stl`. You can simply drag&drop both of them to the `ili window.

Below, you can find an example of a spatial map obtained from `ili for a feature that is localized mainly in the vicinity of rot on the apple.

<img src="img/demo_screenshot.png"/>

## Advanced use-cases

The workflow has many capabilities that you can discover in the documentation embedded into it. Just click on any node, and the description of its role and its parameters will show up in the banner at the right-hand side of the KNIME window. Different nodes don't depend on each other, so you can experiment with different settings and see how workflow output changes.

## Known issues

Some errors can appear in the application log that interrupt workflow execution. A node caused an error will be a left-most node with a clock sign in its center. Errors can happen from time to time depending on the environment at your workstation. Below, you can find a few known errors and workarounds for them.
* An error message about `ConcurrentModificationException` is caused by an internal KNIME error that occurs upon simultaneous access to a single file by several computational nodes.
  * Workaround: if it happened, press `Execute selected and executable nodes` on the KNIME main toolbar, and execution will continue from the point where error occurred.
* `Execute failed: java.lang.NullPointerException` seems like an internal KNIME error.
  * Workaround: Reset a node that produced the error (with right-click menu or `F8` key) and execute it again.
* `Execute failed: Not all chunks finished - check individual chunk branches for details` also caused by malfunctioning during  multithreaded execution of some nodes.
  * Workaround: Re-execute a node that produced the error.
 
## License

The content of this project is licensed under the Apache 2.0 licence, see LICENSE.md.

[![Analytics](https://ga-beacon.appspot.com/UA-73611660-2/optimus/readme)](https://github.com/igrigorik/ga-beacon)
