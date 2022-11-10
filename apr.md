---
# Metadata
title: Analysis and project requirements
author: Krzysztof Dziembała
date: "2022-11-10"

# Pandoc document settings
standalone: true
lang: en-GB
# Pandoc LaTeX variables
geometry: [a4paper, bindingoffset=0mm, inner=30mm, outer=30mm, top=30mm, bottom=30mm]
# documentclass: report
fontsize: 12pt
colorlinks: true
numbersections: true
toc: false
lof: false # List of figures

# header-includes:
  # Remove "Chapter N" from the line above chapter name in report class document
  # I could not include a file for some reason, but this works
  # - |
  #   ````{=latex}
  #   \usepackage{titlesec}
  #   \titleformat{\chapter}
  #     {\normalfont\LARGE\bfseries}{Task \thechapter.}{1em}{}
  #   \titlespacing*{\chapter}{0pt}{3.5ex plus 1ex minus .2ex}{2.3ex plus .2ex}
  #   ````
  # Keep footnote numbering
  # - |
  #   ````{=latex}
  #   \counterwithout{footnote}{chapter}
  #   ````
---

<!-- Allow multiple top-level headers (interpreted as chapters by pandoc) -->
<!-- markdownlint-disable MD025 -->
# Use case diagram

![Use case diagam](img/use_case_diagram.svg)

# Requirements

## Functional

### Loading files (POLSL-SE/whaaale#?)

**Description:** Enumerating available loaders, showing an "Open file" dialog, passing selected file to a relevant loader and catching loader exceptions.

**Result:** Replace image data object in the memory with data returned from loader.

**Side effects:** Switch display mode to monochromatic and first band.

#### MATLAB file loader

A loader which accepts `.mat` files, returns the only three-dimensional variable from such file or one selected by user from a list, if there are multiple such variables.

#### ENVI file loader

A loader accepting ENVI files with headers. Reports compatibility with `.hdr` files. Requires a file with the same name and no `.hdr` extension in the same directory as header. Returns a three-dimensional array with data and if possible wavelength labels for all bands.

> **Note**
> These are the files in AVIRIS datasets

## Non-functional

### Performance

??? (hard to estimate without a prototype)

### Portability

Program must be available for Linux and Windows. It must be possible to run the program without installing it. Program should be distributed as a ZIP archive.\
If possible without developer's license and code signing certificate, the program should be available for macOS.

### Usability

All buttons with icons must have a label visible after hovering them with a mouse cursor.

### Reliability

All errors during file I/O (loading images, exporting curves) must be caught and displayed in a dialog box.

### Code organisation

Code must adhere to the following criteria:

#### Text file format

All text files must be UTF-8 encoded and saved with LF line endings.

#### Python source style

Source code files in Python must be formatted with black.
