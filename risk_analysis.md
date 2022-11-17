---
# Metadata
title: Risk Analysis and Management
author: Team 7RS
date: "2022-11-17"

# Pandoc document settings
standalone: true
lang: en-GB
# Lua filter settings (tables-vrules from chrisaga/lua-filters, `-L tables-rules.lua`)
# https://github.com/chrisaga/lua-filters/tree/tables-vrules/tables-vrules
tables-vrules: true
tables-hrules: true
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
# Risk analysis and management

| Risk          | S | L | Description and action | S' | L' |
| ------------- | - | - | ---------------------- | -- | -- |
| Delay of development. | 8 | 4 | Delays in delivering crucial parts will influence the testing team and release of further components. Keep up with created plan and release components on time. | 1 | 2 |
| Tests will fail. | 5 | 5 | Some of tests will finish with failure.\break Developers should follow planned requirements. | 2 | 1 |
| Team member will get sick. | 5 | 4 | This risk should be accepted. | | |
| Ineffective communication. | 8 | 5 | Lack of communication may occur in team successfully decreases quality of product. Team should be in continuous contact and keep updated. | 3 | 3 |
| Framework or library update. | 7 | 5 | Libraries may become unsupported which will influence program execution.\break Use trusted frameworks and libraries. | 2 | 1 |
| File corruption caused by unexpected circumstances. | 8 | 4 | Delays in application development.\break Save regularly and automatically create backups in different path. | 1 | 1 |
| Code accidentally deleted by a cat which is playing on the keyboard while developer is away. | 6 | 3 | Must start over from a backup file or restore saved changes.\break Keep all cats away from keyboard. Use GIT version control system to avoid such situations. | 1 | 1 |
