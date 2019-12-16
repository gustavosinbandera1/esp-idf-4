# ESP-IDF COMPONENTS

 For structure & modularity, IDF has the concept of "components". Each component is built into a static library as part of the build process.

 The smallest possible component would contain one source file only, as well as a component.mk file to define the component and describe the location of that source file.

 # Build System

This document explains the implementation of the ESP-IDF build system and the concept of “components”. Read this document if you want to know how to organise and build a new ESP-IDF project or component.

## Note

This document describes the CMake-based build system, which is the default since ESP-IDF V4.0. ESP-IDF also supports a legacy build system based on GNU Make, which was the default before ESP-IDF V4.0.


## Concepts

- A “project” is a directory that contains all the files and configuration to build a single “app” (executable), as well as additional supporting elements such as a partition table, data/filesystem partitions, and a bootloader.

- “Project configuration” is held in a single file called sdkconfig in the root directory of the project. This configuration file is modified via idf.py menuconfig to customise the configuration of the project. A single project contains exactly one project configuration.
  
- An “app” is an executable which is built by ESP-IDF. A single project will usually build two apps - a “project app” (the main executable, ie your custom firmware) and a “bootloader app” (the initial bootloader program which launches the project app).
- “components” are modular pieces of standalone code which are compiled into static libraries (.a files) and linked into an app. Some are provided by ESP-IDF itself, others may be sourced from other places.
- “Target” is the hardware for which an application is built. At the moment, ESP-IDF supports only one target, esp32.


# Some things are not part of the project:

“ESP-IDF” is not part of the project. Instead it is standalone, and linked to the project via the IDF_PATH environment variable which holds the path of the esp-idf directory. This allows the IDF framework to be decoupled from your project.
The toolchain for compilation is not part of the project. The toolchain should be installed in the system command line PATH.


# Example Project

```
- hello_world/
             - CMakeLists.txt
             - sdkconfig
             - components/ - component1/ - CMakeLists.txt
                                         - Kconfig
                                         - src1.c
                           - component2/ - CMakeLists.txt
                                         - Kconfig
                                         - src1.c
                                         - include/ - component2.h
             - main/       - CMakeLists.txt
                           - src1.c
                           - src2.c

             - build/

```
### This example “hello_world” contains the following elements:

- A top-level project CMakeLists.txt file. This is the primary file which CMake uses to learn how to build the project; and may set project-wide CMake variables. It includes the file /tools/cmake/project.cmake which implements the rest of the build system. Finally, it sets the project name and defines the project.

- ```sdkconfig``` project configuration file. This file is created/updated when idf.py menuconfig runs, and holds configuration for all of the components in the project (including ESP-IDF itself). The “sdkconfig” file may or may not be added to the source control system of the project.
  
- Optional “components” directory contains components that are part of the project. A project does not have to contain custom components of this kind, but it can be useful for structuring reusable code or including third party components that aren’t part of ESP-IDF. Alternatively, EXTRA_COMPONENT_DIRS can be set in the top-level CMakeLists.txt to look for components in other places. See the renaming main section for more info. If you have a lot of source files in your project, we recommend grouping most into components instead of putting them all in “main”.

- ```main``` directory is a special component that contains source code for the project itself. “main” is a default name, the CMake variable COMPONENT_DIRS includes this component but you can modify this variable.

- Component directories each contain a component CMakeLists.txt file. This file contains variable definitions to control the build process of the component, and its integration into the overall project. See Component CMakeLists Files for more details.

- Each component may also include a Kconfig file defining the component configuration options that can be set via menuconfig. Some components may also include Kconfig.projbuild and project_include.cmake files, which are special files for overriding parts of the project.

# Component CMakeLists Files

Each project contains one or more components. Components can be part of ESP-IDF, part of the project’s own components directory, or added from custom component directories.

A component is any directory in the COMPONENT_DIRS list which contains a CMakeLists.txt file.

# Minimal Component CMakeLists

The minimal component CMakeLists.txt file simply registers the component to the build system using idf_component_register:

```
idf_component_register(SRCS "foo.c" "bar.c"
                       INCLUDE_DIRS "include"
                       REQUIRES mbedtls)
```