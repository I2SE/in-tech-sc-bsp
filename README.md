# Yocto Environment for Tarragon and EVAcharge SE platforms

This is a wrapper repository for in-tech smart charging GmbH Yocto BSP and distribution open-source layers.  
For problems and inquiries: https://tickets.in-tech-smartcharging.com/servicedesk

## Table of Contents
1. [Introduction](#introduction)
2. [Background](#background)  
2.1 [Layers](#layers)  
2.2 ["Wrapper" Repository](#wrapper)  
3. [Build with Yocto](#building)   
3.1 [System Requirements](#SystemRequirements)  
3.2 [Setting up the Yocto build environment](#Setting)  
3.3 [Adding or removing layers](#addorremove)  
3.4 [Building an Image](#build)


## Introduction <a name="introduction"></a>

This document helps you to get started with creating a Linux image based on board support packages (BSP) of Tarragon & EVAcharge SE; hardware platforms offered by in-tech smart charging GmbH. It defines what the layers included in this Yocto Project are, and how you can use them to create a basic Linux distribution, which you can then extend by adding further packages specific to your application.

If you are new to Yocto, it is recommended to read the [Yocto Overview and Concepts Manual](https://docs.yoctoproject.org/overview-manual/index.html). To get a quick introduction about Yocto, this [Software Overview](https://www.yoctoproject.org/software-overview) might be helpful. For further documentations about the Yocto Project, including information about dealing with BSP layers and working with the Yocto Project's build system **BitBake**, check the [Yocto Project Documentation](https://docs.yoctoproject.org/).

## Background <a name="background"></a>
### Layers <a name="layers"></a>
As the Yocto Project is based on the concepts of [layers](https://docs.yoctoproject.org/dev-manual/common-tasks.html#understanding-and-creating-layers), the following table lists all the layers used to create a basic distribution based on in-tech smart charging’s charge control platforms Tarragon and EVAcharge SE.

| Layer | Description | Repository |
|--|--|--|
| meta-in-tech-sc | BSP layer for Tarragon & EVAcharge SE | https://github.com/I2SE/meta-in-tech-sc |
| meta-in-tech-sc-distro | Distribution adaptations layer | https://github.com/I2SE/meta-in-tech-sc-distro |
| meta-freescale | Layer containing NXP hardware support metadata | https://git.yoctoproject.org/cgit/cgit.cgi/meta-freescale |
| meta-openembedded | Collection of layers to supplement OE-Core with additional packages | https://github.com/openembedded/meta-openembedded |
| meta-rauc| Layer controlling and performing secure software updates for embedded Linux | https://github.com/I2SE/rauc |
| poky | Build tool and metadata included in a reference distribution | https://github.com/I2SE/poky |

This layering approach increases flexibility to expand your project. You can add layers, which in turn would add packages essential for the distribution you want to build. Layers are available usually as repositories. Information on how to include or remove layers will be given in [Section 3.3](#addorremove). Note that no charging capabilities will be included in the Linux distribution created by this setup.

### "Wrapper" Repository <a name="wrapper"></a>
This "wrapper" repository has been created to facilitate downloading the above-mentioned layers/repositories on your local machine. It contains a manifest file called `default.xml` that contains information about the repositories representing the layers needed to build a distribution, and which branches to be checked out from these repositories. The process of extracting information out of the manifest file and checking out the branches of the mentioned repositories is done by using `repo` commands. Installation and usage of the `repo` utility will be explained in [Section 3.2](#Setting). The manifest file for this repository looks like the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <default sync-j="4" revision="thud"/>

  <!-- remote repository definitions -->
  <remote fetch="https://git.yoctoproject.org/git" name="yocto"/> <!-- This represents a link to a repository, and it will have a name for further usage -->
  <remote fetch="https://github.com/openembedded" name="oe"/>
  <remote fetch="https://github.com/I2SE" name="i2se"/>
  <remote fetch="https://github.com/I2SE" name="rauc"/>

  <!-- project definitions -->
  <project remote="i2se"    revision="582cfbc66925a3f7879bd2289835352b342d8985" name="poky"                   path="source"/>
  <project remote="yocto"   revision="b73854c078b0a174613135b60da3377a1055f477" name="meta-freescale"         path="source/meta-freescale"/>
  <project remote="oe"      revision="9b3b907f30b0d5b92d58c7e68289184fda733d3e" name="meta-openembedded"      path="source/meta-openembedded"/>
  <project remote="i2se"    revision="thud"                                     name="meta-in-tech-sc"        path="source/meta-in-tech-sc"/>
  <project remote="i2se"    revision="thud"                                     name="meta-in-tech-sc-distro" path="source/meta-in-tech-sc-distro"/>
  <project remote="rauc"    revision="17599be65f6a5eabe6e4a246767c06dc4507f21a" name="meta-rauc"              path="source/meta-rauc"/>
  <project remote="i2se"    revision="thud"                                     name="in-tech-sc-bsp"         path="in-tech-sc-bsp">
    <linkfile dest="build/conf" src="conf"/>
  </project>

</manifest>
```

The attributes `revision` and `path` represent the source branch and the destination where the branch content will be placed on the local machine, respectively. You can set the `revision` to be either a branch name, by which you can continuously receive branch updates, or a particular commit to the repository. The tag `linkfile` links two folders together. It has two attributes; one is `src` which is a folder in the repository, and the other is `dest` representing the destination folder in the local machine, which will always have the content of the source folder i.e., mirror it.

Apart from the manifest file, the repository has also a configuration folder. This folder contains a `local.conf` file, that contains some variables and machine configurations you can alter to affect the resulting distribution, and a `bblayers.conf` file that gives information about which layers will be included for building. Adding or removing layers can be done through this file, provided that the added layer, e.g., the cloned repository, is found on the local machine in the path described in this file. 

## Build with Yocto <a name="building"></a>
### System Requirements <a name="SystemRequirements"></a>
Some packages are required by the build host to be able to cover all build scenarios using the Yocto Project. In this section of the [Yocto Reference Manual](https://docs.yoctoproject.org/ref-manual/system-requirements.html#required-packages-for-the-build-host) you can find some helpful instructions based on the Linux distribution you are using. If you are using hosts other than Linux, this section of the [Yocto Project Development Tasks Manual](https://docs.yoctoproject.org/dev-manual/start.html#preparing-the-build-host) can help you setting up your host system for using Yocto. 

### Setting up the Yocto build environment <a name="Setting"></a>
To be able to build an image with Yocto, the following setup should be followed: 

1. To make use of the manifest file, you have to install `repo` to get your Yocto environment ready. The `repo` utility was originally created to ease Android development. It makes it easy to reference several Git repositories within a top-level project, which you can then clone to your local machine all at once. 

```bash
mkdir ~/bin
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

You need to also make sure that `~/bin` is added to your `PATH` variable (Usually the directory is added automatically in Ubuntu).

```bash
echo 'export PATH="$PATH":~/bin' >> ~/.bashrc
```

2. Now you can use the `repo` tool to check out all the repositories listed in the manifest file. 

```bash
mkdir yocto
cd yocto
repo init -u https://github.com/I2SE/in-tech-sc-bsp -b thud
repo sync
```

After the command `repo sync` is executed, you should be able to find three folders in the created `yocto` directory:
1. `source`: Where all the repositories representing the layers are cloned.  
2. `in-tech-sc-bsp`: A clone of the 'wrapper' repository containing the manifest file and configurations folder.
3. `build`: A link to the `conf` folder in `in-tech-sc-bsp`

### Adding or removing layers <a name="addorremove"></a>
Adding a layer can be done either:

**Manually**
1. Download the layer as a tarball or by cloning a repository.
2. Copy the folder where the layer is downloaded.
3. Paste it in the `yocto/source` folder.
4. Edit the `yocto/build/conf/bblayers.conf` file to include the new layer with the right path.

or **Automatically**
1. Edit the manifest file `in-tech-sc-bsp/default.xml`, so that it contains information about the source of the new layer
2. Issue the command `repo sync`. Note that all other layers will be synced, so any unsaved changes to your layers will be lost.
3. Edit the `yocto/build/conf/bblayers.conf` file to include the new layer with the right path.

You can then execute the command `bitbake-layers show-layers` to make sure that the new layer is now included.

To remove a layer, you can simply alter the `bblayers.conf` file by removing the layer path to make sure that the build system does not consider this layer while generating an image.

### Building an image <a name="build"></a>
To rightly set configuration related to the hardware platforms Tarragon and EVAcharge SE, the following table gives you insights about the images you can build:

| **`MACHINE`** | **`PROJECT`** | **`CUSTOMER`** | Resulting image |
|--|--|--|--|
| `tarragon` | `bsp` | `""` | Basic BSP image for Tarragon |
| `tarragon` | `bsp` | `developer`[^1] | BSP image for Tarragon with additional developer packages |
| `evachargese` | `bsp` | `""` | Basic BSP image for EVAcharge SE |
| `evachargese` | `bsp` | `developer` | BSP image for EVAcharge SE with additional developer packages |

For building an image, you would need to do the following:
1. Set the configurations for your build as mentioned in the table above. You can either:  
  - Execute the following commands to e.g., set the machine to `tarragon` and project to `bsp`:  
```bash
export MACHINE=tarragon
export PROJECT=bsp
export BB_ENV_EXTRAWHITE="PROJECT MACHINE"
```
  - Edit `yocto/build/conf/local.conf` directly. e.g., `MACHINE=...`.
2. Execute `source yocto/source/oe-init-build-env build` which initializes the build environment and changes the directory to `yocto/build`.
3. Execute `bitbake core-image-minimal` to build the image.

The resulting image will be found in `yocto/build/tmp/deploy/image/<machine>`.

[^1]: to be able to build a developer image, check the respective documentation in `local.conf` 
