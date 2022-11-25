---
# Metadata
title: Software Design
author: Team 7RS
date: "2022-11-24"

# Pandoc document settings
standalone: true
lang: en-GB
# Pandoc LaTeX variables
geometry: [a4paper, bindingoffset=0mm, inner=30mm, outer=30mm, top=30mm, bottom=30mm]
# documentclass: report
fontsize: 12pt
colorlinks: true
numbersections: true
toc: true
toc-depth: 4
lof: false # List of figures

header-includes:
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
  # Wrap long source code lines
  - |
    ````{=latex}
    \usepackage{fvextra}
    \DefineVerbatimEnvironment{Highlighting}{Verbatim}{breaklines,commandchars=\\\{\}}
    ````
  # Make \paragraph (4-th level header) a header instead of a paragraph
  - |
    ````{=latex}
    \usepackage{titlesec}
    \titleformat{\paragraph}
    {\normalfont\normalsize\bfseries}{\theparagraph}{1em}{}
    \titlespacing*{\paragraph}
    {0pt}{3.25ex plus 1ex minus .2ex}{1.5ex plus .2ex}
    ````
---

<!-- markdownlint-configure-file
{
  "MD024": { "allow_different_nesting": true }
}
-->

<!-- Allow multiple top-level headers (interpreted as chapters by pandoc) -->
<!-- markdownlint-disable MD025 -->
# Classes

## UML diagram

<!-- mermaid-filter (npm i -g mermaid-filter) has to be installed and pandoc should be executed with `-F mermaid-filter[.cmd]` (.cmd on Windows) -->
```{.mermaid format=pdf width=1200}
classDiagram
  class Loader {
    -list~AbstractFileLoader~ loaders
    -QWidget parent
    -supported_types() list~str~
    +open_file() HsImage
    -show_error(Exception e)
    -load_file(str path) HsImage
  }

  class AbstractFileLoader {
    <<Abstract>>
    +str FILE_FILTER_NAME*
    +list~str~ EXTENSIONS*
    +load_file(str path)* HsImage
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
    -Loader loader
    -ApplicationState state
    -HsImage|None image
    -tuple~int, int~|None start_position
    -ImageMode image_mode
    -int band_mono
    -int band_r
    -int band_g
    -int band_b
    -ImagePreview preview
    -SpectralViewer viewer

    +on_mode_change()
    +on_band_change()
    +on_mouse_down(tuple~int, int~ coordinates)
    +on_mouse_up(tuple~int, int~ coordinates)
  }
  MainWindow --> ApplicationState
  MainWindow --> ImageMode
  HsImage "0..1" --* "1" MainWindow
  Loader "1" --* "1" MainWindow
  ImagePreview "0..1" --* "1" MainWindow
  MainWindow --|> QMainWindow

  %% bpp = bits per pixel, usually 8 or 16
  %% labels should be a sequence 1..=N if not provided with data
  class HsImage {
    -ndarray data
    +int bpp
    +list~str~ labels
    +get_pixel(int x, int y) ndarray
    +get_area(tuple~int, int~ p1, tuple~int, int~ p2) ndarray
    +get_similar(tuple~int, int~ base, float threshold) ndarray
    +get_band(int idx) ndarray
    +get_RGB_bands(int r_idx, int g_idx, int b_idx) ndarray
  }

  class ImagePreview {
    -QPixmap bitmap
    -MainWindow parent
    -QRubberBand|None rubber_band
    -on_mouse_down(QMouseEvent event)
    -on_mouse_up(QMouseEvent event)
    -on_mouse_move(QMouseEvent event)
    +draw_rubber_band_from(tuple~int, int~ start)
    +clear_rubber_band()
    +render_single(ndarray band, int bpp)
    +render_rgb(ndarray rgb_bands, int bpp)
    +render_similar(ndarray band, int bpp, ndarray mask)
  }
  ImagePreview --> MainWindow : dispatches mouse events

  class MatlabLoader {
    -load_hdf5(file) HsImage
    -load_mat_b72(file) HsImage
  }

  class SpectralViewer {
    -MainWindow parent
    -QChart chart
    -QChartView chart_view
    -AreaValues|PixelValues|None data
    -list~str~ labels
    +from_pixel(ndarray pixel)
    +from_area(ndarray area)
    +update_labels(list~str~ labels)
    +clear()
    +export_csv()
    +export_png()
  }
  SpectralViewer "0..1" --* "1" MainWindow

  AreaValues "0..1" --o "1" SpectralViewer
  PixelValues "0..1" --o "1" SpectralViewer
  AreaValues .. PixelValues : or

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

Derived from `QMainWindow`. Contains UI setup, most UI elements and event handlers for buttons.

Stores:

- loaded image
- [state](#state-diagram)
- image mode
- selected bands
  - mono
  - red
  - green
  - blue
- tolerance for the *magic wand* functionality
- starting position of area selection

Event handlers:

- open file menu action - request loading file from [`Loader`](#loader)\
  After the file is loaded:
  1. Save the [`HsImage`](#hsimage)
  2. Set state to `IMAGE_LOADED`
  3. Reset selected bands to default values
  4. Clear [`SpectralViewer`](#spectralviewer) or update it with the whole image and update its labels
  5. Clear the rubber band on [`ImagePreview`](#imagepreview)
  6. Render the new image in [`ImagePreview`](#imagepreview)
- mode selection - display appropriate band settings and update the preview
- band change - save the new value and update the preview
- ESC button clicked - set state to `IMAGE_LOADED` (or `NO_IMAGE`) and clear the rubber band
- select pixel/area/similar - update the state according to the clicked button

Actions triggered by [`ImagePreview`](#imagepreview):

- mouse down - ignore if not in `SELECT_AREA_FIRST` state, save `start_position`, request drawing a rubber band and change state to `SELECT_AREA_SECOND`
- mouse up - action depends on current state:
  - `SELECT_PX` - get selected pixel and update [`SpectralViewer`](#spectralviewer)
  - `SELECT_AREA_SECOND` - get area between `start_position` and received coordinates and update [`SpectralViewer`](#spectralviewer)
  - `SELECT_SIMILAR` - get similar pixels, change image mode to `SIMILAR` and update [`ImagePreview`](#imagepreview)

  After any of those actions state must be restored to `IMAGE_LOADED`.

### `Loader`

Provides an interface for loading files. Manages errors during file loading (I/O, invalid format, expectations not met, etc.).
Has a list of available file loaders.

#### Notes for implementer

1. Use static `QFileDialog` methods - it is simpler to implement. Example:

    ```python
    file_path, used_filter = QFileDialog.getOpenFileName(
        parent,
        "Open image",
        "",
        "Matlab file (*.mat);;ENVI .hdr labelled image (*.hdr);;All supported types (*.mat *.hdr)",
    )
    ```

2. Keep (or get as a function parameter) a reference to parent, to block the parent (main) window, while a dialog (open file, file loader *settings* or error) is open.
3. Error messages can be displayed using `QMessageBox` with static `.warning` or property-based API if informative or detailed text should be set.

### `AbstractFileLoader`

An [abstract base class](https://docs.python.org/3/library/abc.html) for specialised file loaders. Each loader must have a getter for a friendly *category* name and a list of supported extensions. `load_file(path: str) -> HsImage` abstract method provides an universal interface for loading files.

### `MatlabLoader`

Derived from [`AbstractFileLoader`](#abstractfileloader). Supports `.mat` files. If multiple three-dimensional variables are available prompt the user for selection.

#### Notes for implementer

1. `QInputDialog` can be used for variable selection:

    ```python
    variable_names = ["data", "something", "a_name"]
    selected_var, ok = QInputDialog.getItem(
        parent,
        "Select variable",
        "Variable containing image data",
        variable_names,
        editable=False,
    )
    ```

2. For MAT-files <= 7.2 use [`scipy.io.loadmat`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.io.loadmat.html)\
   For 7.3 use [`h5py`](https://docs.h5py.org/en/stable/index.html) (a HDF5 library for Python). `h5py` is not available for Python 3.11 yet.

3. **Warning:** Some datasets have negative values instead of using unsigned integers.

### `ENVILoader`

Derived from [`AbstractFileLoader`](#abstractfileloader). Supports ENVI `.hdr` labelled files. Should report compatibility with `.hdr` files, but a file with the same name, but no `.hdr` extension must exist in the same directory.

#### Notes for implementer

1. ENVI files can be loaded using [`GDAL`](https://gdal.org/api/python_bindings.html), but its installation may be tricky. Windows packages can be found [here](https://www.lfd.uci.edu/~gohlke/pythonlibs/#gdal), for Linux it probably will have to be built from source.

### `HsImage`

Stores image data:

- raw pixel data as a 3D array [height, width, bands]
- bits per pixel
- band labels

If band labels are not provided in the image file a sequence `1..=bands` should be used instead.

Must provide:

- pixel data given its coordinates
- subarray with pixels bounded by given coordinates
- a mask with pixels similar to one with given coordinates and a threshold
- a single band of the image given its index
- three bands in specified order given their indices

### `ImagePreview`

Displays the image preview using specified mode. Proxies events to [`MainWindow`](#mainwindow) after translating mouse position to pixel coordinates. Draws a `QRubberBand` when selecting an area.

*Optional:* Manages zoom and panning.

### `AreaValues` and `PixelValues`

Are derived from [`TypedDict`](https://docs.python.org/3/library/typing.html#typing.TypedDict) ([PEP 589](https://peps.python.org/pep-0589/)). Both store data displayed in [`SpectralViewer`](#spectralviewer) - either *aggregate values* (`AreaValues` - avg, min, max, quartiles) or a single pixel value (`PixelValue`). Their properties are vectors (1D `ndarray`s).

### `SpectralViewer`

Manages the spectral curve chart. Chart is labeled using stored labels, which should be updated when a file is loaded. When created from a pixel or an area it should calculate appropriate values, store them as [`PixelValues`](#areavalues-and-pixelvalues) or [`AreaValues`](#areavalues-and-pixelvalues) and update the chart.\
Two methods for exporting must be provided:

- exporting to CSV - simply write [`PixelValues` or `AreaValues`](#areavalues-and-pixelvalues) and labels to CSV
- exporting as PNG - render the `chart` or `chart_view` to a `QPixmap` and save it

**Note:** If needed exporting as JPEG can also be provided just by allowing to save `QPixmap` with `jpg`/`jpeg` extension.

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

## User interactions

### Open image

<!-- Requires "break" introduced in mermaid 9.1.2. Updated mermaid-filter is available on GitHub Packages as @krzysdz/mermaid-filter. -->
```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Loader
  participant ML as MatlabLoader
  participant ENVI as ENVILoader
  participant Img as HsImage
  participant Prev as ImagePreview
  participant Chart as SpectralViewer

  User->>Main: Open file
  Main->>+Loader: Open file
  Loader-->Loader: Get file loaders

  par Matlab loader details
    Loader->>+ML: Get filter name
    ML-->>-Loader: Return filter name
    Loader->>+ML: Get supported extensions
    ML-->>-Loader: Return supported extensions
  and ENVI loader details
    Loader->>+ENVI: Get filter name
    ENVI-->>-Loader: Return filter name
    Loader->>+ENVI: Get supported extensions
    ENVI-->>-Loader: Return supported extensions
  end

  Loader->>+User: Show "Open file" dialog
  User->>-Loader: Select file

  alt File is supported by MatlabLoader
    Loader->>+ML: Load file
    alt Multiple 3D variables in file
      ML->>+User: Show variable picker dialog
      User->>-ML: Select a variable
    end
    ML-->Img: Create
    ML-->>-Loader: Return HsImage
  else File is supported by ENVILoader
    Loader->>+ENVI: Load file
    break If data file is missing
      ENVI-->Loader: Throw an exception
      Loader->>User: Display an error message
    end
    ENVI-->Img: Create
    ENVI-->>-Loader: Return HsImage
  end

  Loader-->>-Main: Return HsImage
  Main-->Main: Reset band setting
  Main-->Main: Set state to IMAGE_LOADED
  Main->>Prev: Clear rubber band
  Main->>+Prev: Update image
  Prev-->>-User: Show new image
  Main->>Chart: Clear chart
  Main->>Chart: Update labels
```

### Display a single band

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Img as HsImage
  participant Prev as ImagePreview

  note over Main,Prev: Image must already be loaded

  User->>Main: Display a single band
  Main-->Main: Set mode to MONO
  Main->>+Img: Get band
  Img-->>-Main: Return image band
  Main->>+Prev: Render a single band
  Prev-->>-User: Display image preview

  User->>Main: Change band for MONO mode
  Main->>+Img: Get band
  Img-->>-Main: Return image band
  Main->>+Prev: Render a single band
  Prev-->>-User: Display updated image preview
```

### Display fake-colored image

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Img as HsImage
  participant Prev as ImagePreview

  note over Main,Prev: Image must already be loaded

  User->>Main: Display fake-colored image
  Main-->Main: Set mode to RGB
  Main->>+Img: Get RGB bands
  Img-->>-Main: Return bands
  Main->>+Prev: Render an RGB image
  Prev-->>-User: Display image preview

  User->>Main: Change a[n] R/G/B band
  Main->>+Img: Get RGB bands
  Img-->>-Main: Return bands
  Main->>+Prev: Render an RGB image
  Prev-->>-User: Display updated image preview
```

### Display curve for a single pixel

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Img as HsImage
  participant Prev as ImagePreview
  participant Chart as SpectralViewer

  note over Main,Chart: Image must already be loaded

  User->>Main: Curve for a pixel
  Main-->Main: Set state to SELECT_PX
  User->>+Prev: Clicks (mouse up) on an image
  Prev->>-Main: Mouse up pixel coordinates
  activate Main
  Main-->Main: Set state to IMAGE_LOADED
  Main->>+Img: Get pixel
  Img-->>-Main: Return pixel data
  Main->>-Chart: Create a chart from pixel data
  activate Chart
  Chart-->>-User: Display the updated chart
```

### Display curve for a region

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Img as HsImage
  participant Prev as ImagePreview
  participant Chart as SpectralViewer

  note over Main,Chart: Image must already be loaded

  User->>Main: Curve for an area
  Main-->Main: Set state to SELECT_AREA_FIRST
  User->>+Prev: Clicks (mouse down) on an image
  Prev->>-Main: Mouse down pixel coordinates
  activate Main
  Main-->Main: Save start position
  Main-->Main: Set state to SELECT_AREA_SECOND
  Main->>-Prev: Start drawing a rubber band
  loop Until mouse up
    User->>+Prev: Moves the cursor
    Prev-->>-User: Update rubber band shape
  end
  User->>+Prev: Clicks (mouse up) on an image
  Prev->>-Main: Mouse up pixel coordinates
  activate Main
  Main->>+Img: Get area data
  Img-->>-Main: Return area data
  Main->>-Chart: Create a chart from area
  activate Chart
  Chart-->>-User: Display the updated chart
```

### Select similar pixels

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Img as HsImage
  participant Prev as ImagePreview

  note over Main,Prev: Image must already be loaded

  User->>Main: Select similar pixels
  Main-->Main: Set state to SELECT_SIMILAR
  opt Adjusting threshold
    User->>Main: Change threshold
  end
  User->>+Prev: Clicks (mouse up) on an image
  Prev->>-Main: Mouse up pixel coordinates
  activate Main
  Main-->Main: Set state to IMAGE_LOADED
  Main-->Main: Set mode to SIMILAR
  Main->>+Img: Get band
  Img-->>-Main: Return image band
  Main->>+Img: Get similar pixels
  Img-->>-Main: Return similarity map
  Main->>+Prev: Render a single band with similar pixels
  deactivate Main
  Prev-->>-User: Display updated image preview
```

### Export spectral curve(s)

#### Export as CSV

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Chart as SpectralViewer
  participant File as File-like object

  note over Main,File: A chart must already exist

  User->>Main: Export chart as CSV
  Main->>+Chart: Export as CSV
  Chart->>+User: Show "Save file" dialog
  User->>-Chart: Select file
  Chart-->File: Open for writing
  break Opening for writing fails
    File-->Chart: Throw an exception
    Chart->>User: Display an error message
  end
  Chart->>File: Write data
  Chart->>File: Close
  deactivate Chart
```

#### Export as PNG

```{.mermaid format=pdf width=1200}
sequenceDiagram
  actor User
  participant Main as MainWindow
  participant Chart as SpectralViewer
  participant Pixmap as QPixmap

  note over Main,Pixmap: A chart must already exist

  User->>Main: Export chart as PNG
  Main->>+Chart: Export as PNG
  Chart->>+User: Show "Save file" dialog
  User->>-Chart: Select file
  Chart-->Pixmap: Render to QPixmap
  Chart->>+Pixmap: Save to file
  break Saving QPixmap fails
    Pixmap-->>-Chart: Return false
    note left of Pixmap: QPixmap does not return detailed errors
    Chart->>User: Display an error message
  end
  deactivate Chart
```
