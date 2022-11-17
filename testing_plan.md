---
# Metadata
title: Whaaale Testing Plan
author: Team 7RS
date: "2022-11-16"

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
  #     {\normalfont\LARGE\bfseries}{\thechapter.}{1em}{}
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
# Automatic tests

1. Checking if all icons have labels.

   Procedure:
   - Hovering over all available icons.

   Expected outcome:
   - Short description of all icons.

2. Loading several files and checking if loaded correctly

   Prerequisites:
   - Choosing a desired folder path with files.

   Procedure:
   - Load .mat file,
   - Load .hdr file(both data and header).

   Expected outcome:
   - Files should be loaded without errors.

# Manual tests

1. Check if hyperspectral image is loading to program:

   Prerequisites:
   - Prepare .mat file,
   - Prepare .hdr file one for data and other for header.

   Procedure:
   - Load .mat file,
   - Load .hdr file (both data and header).

   Expected outcome:
   - File should be loaded with no errors.

2. Check if program display single band:

   Prerequisites:
   - Loaded image file.

   Expected outcome:
   - Monochromatic image should be displayed.

3. Check program displays selected color band image:

   Prerequisites:
   - Loaded image file.

   Expected outcome:
   - Image should be displayed in the selected color band.

4. Check if point for spectral curve can be selected:

   Prerequisites:
   - Loaded image file.

   Expected outcome:
   - Spectral curve should be modified according to the selected pixel.

5. Check if is possible to export the curve:

   Prerequisites:
   - Loaded image file.

   Expected outcome:
   - Exported CSV or PNG file containing curve.

6. Find similar pixels:

   Prerequisites:
   - Loaded image file.
   Expected outcome:
   - All pixels similar to selected with tolerance are displayed.

# Non functional tests

- Check if program works on different environments ie. Mac and Windows
- Check size of program (max 500 MB)
- Ensure that UI is clear and readable

# Timetable of tests

| Stage | Dates |
| - | - |
| Test planning: | 10.11.2022 - 17.11.2022 |
| Iteration 1 of functional testing: | 18.11.2022 - 25.11.2022 |
| Development team correction: | 25.11.2022 - 29.11.2022 |
| Iteration 2 of functional testing: | 29.11.2022 - 02.12.2022 |
| Release to production: | 06.12.2022 |

<!-- mermaid-filter (npm i -g mermaid-filter) has to be installed and pandoc should be executed with `-F mermaid-filter[.cmd]` (.cmd on Windows), make sure that rsvg-convert is in PATH -->
```{.mermaid format=svg}
gantt
   title Test schedule
   dateformat YYYY-MM-DD
   todayMarker off
   %% YYYY-MM-DD overlaps (mermaid-js/mermaid#1301)
   axisFormat %b %e

   section Test Planning
   Planning : active,2022-11-10,2022-11-18

   section Iteration 1
   Testing : 2022-11-18,2022-11-26

   section Corrections
   Fixing bugs : 2022-11-25,5d

   section Iteration 2
   Testing : 2022-11-29,4d

   section Release
   Release : 2022-12-06,1d
```
