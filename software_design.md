---
# Metadata
title: Software Design
author: Team 7RS
date: \today

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
# Classes

## UML diagram

<!-- mermaid-filter (npm i -g mermaid-filter) has to be installed and pandoc should be executed with `-F mermaid-filter[.cmd]` (.cmd on Windows) -->
```{.mermaid format=pdf}
classDiagram
  class Loader {
    list~AbstractFileLoader~ loaders
    supported_types() list~str~
    open_file() HsImage
  }

  class AbstractFileLoader {
    <<Abstract>>
    str FILE_FILTER$
    MainWindow parent
    load_file() HsImage
    show_error(Exception e)
  }

  %% from enum import IntEnum
  class ApplicationState {
    <<Enumeration>>
    NO_IMAGE
    IMAGE_LOADED
    SELECT_PX
    SELECT_AREA_FIRST
    SELECT_AREA_SECOND
    SELECT_SIMILAR
  }
  IntEnum <|-- ApplicationState

  class ImageMode {
    <<Enumeration>>
    MONO
    RGB
    SIMILAR
  }
  IntEnum <|-- ImageMode

  class MainWindow {
    Loader loader
    ApplicationState state
    HsImage|None image
    tuple~int, int~|None start_position
    ImageMode image_mode
    int band_mono
    int band_r
    int band_g
    int band_b
    ImagePreview preview
    SpectralViewer viewer

    on_mode_change()
    on_band_change()
  }
  MainWindow --> ApplicationState
  MainWindow --> ImageMode
  HsImage "0..1" --* "1" MainWindow
  ImagePreview "0..1" --* "1" MainWindow
  Loader "1" --* "1" MainWindow

  %% bpp = bits per pixel, usually 8 or 16
  %% labels should be a sequence 1..=N if not provided with data
  class HsImage {
    ndarray data
    int bpp
    list~str~ labels
    get_pixel(int x, int y) ndarray
    get_area(tuple~int, int~ p1, tuple~int, int~ p2) ndarray
  }

  class ImagePreview {
    QPixmap bitmap
    MainWindow parent
    QRubberBand|None rubber_band
    on_mouse_down(QMouseEvent event)
    on_mouse_up(QMouseEvent event)
    on_mouse_move(QMouseEvent event)
    draw_rubber_band_from(tuple~int, int~ start)
    clear_rubber_band()
  }

  class MatlabLoader {
    list~str~ variables
    show_select_var()
  }

  class SpectralViewer {
    MainWindow parent
    QChartView chart_view
    AreaValues|PixelValues data
    int max_data
    from_pixel(ndarray pixel)
    from_area(ndarray area)
    update_labels(list~str~ labels)
    set_max(int max)
    clear()
    export_csv()
    export_png()
  }
  SpectralViewer "0..1" --* "1" MainWindow
  AreaValues --o SpectralViewer
  PixelValues --o SpectralViewer

  class AreaValues {
    ndarray avg
    ndarray min
    ndarray max
    ndarray quartile_low
    ndarray quartile_high
  }
  TypedDict <|-- AreaValues

  class PixelValues {
    ndarray values
  }
  TypedDict <|-- PixelValues

  ENVILoader ..|> AbstractFileLoader : Implements
  MatlabLoader ..|> AbstractFileLoader : Implements
  AbstractFileLoader "*" --o "1" Loader
  AbstractFileLoader --> HsImage : creates
```

## Class description

### `MainWindow`

Contains UI setup, most UI elements and event handlers

### `ImagePreview`

Proxies events to `MainWindow` after translating mouse position to pixel coordinates.

*Optional:* Manages zoom and panning.

# High level overview

## State diagram

```{.mermaid format=pdf}
stateDiagram-v2
  state "No image" as no_img
  [*] --> no_img

  state "Image loaded" as img
  note left of img
    How it is displayed depends
    on other settings.
  end note
  no_img --> img : Load image (successfully)

  state "Select a pixel active" as s_px
  img --> s_px : Select a pixel clicked
  s_px --> img : ESC button clicked

  state SelectArea {
    state "Waiting for first coordinate" as s_area_first
    [*] --> s_area_first

    state "Waiting for second coordinate" as s_area_second
    note right of s_area_second
      Draw a QRubberBand from
      the first coordinate to
      the currently hovered pixel.
      Track using mouseMoveEvent.
    end note
    s_area_first --> s_area_second : mousePressEvent

    s_area_second --> [*] : mouseReleaseEvent
  }
  img --> SelectArea : Select an area clicked
  SelectArea --> img : ESC button clicked

  state "Update spectral curve" as update_curve
  s_px --> update_curve : mouseReleaseEvent
  SelectArea --> update_curve
  update_curve --> img

  state "Select similar pixels active" as sim
  img --> sim : Select similar pixels clicked
  sim --> img : ESC button clicked

  SelectArea --> s_px : Select a pixel clicked
  sim --> s_px : Select a pixel clicked
  s_px --> SelectArea : Select an area clicked
  sim --> SelectArea : Select an area clicked
  s_px --> sim : Select similar pixels clicked
  SelectArea --> sim : Select similar pixels clicked
```
