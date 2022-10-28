---
title: Software Engineering 2022
subtitle: Task 4
author: Michał Kawulok, D.Sc.
---

# **Project name:** Hyperspectral data viewer

## Background

Hyperspectral images are composed of multiple channels, each of which presents a different spectral band. Such images can be visualized in many different ways - either as individual bands or as a fake-colored image (with selected bands assigned as red, green and blue channels). On the other hand, the spectral curves can be displayed that show the spectral signatures of a single pixel, selected region, or the entire image.

## Task description

The main goal of this tool is to visualize the selected hyperspectral data cube in many different ways. It is required that the tool meets the following:

1. Displaying a single selected band.
2. Displaying a fake-colored image with selected bands used as individual color channels.
3. Rendering a spectral curve for a selected pixel or region. When rendering a curve for multiple pixels, show the average curve, minimum, maximum, as well as the quartile curves.
4. Make it possible to select multiple pixels based on the spectral angle distance from a single pixel (with the maximum angle as a parameter).
5. Make the curves exportable as csv or png.

## Additional notes

1. Ad 4. - use MSE (calculated as $\frac{1}{n}\sum_{i = 1}^{n}{(y_i - X_i)^2}$, where $n$ is the number of bands, $y_i$ is the tested pixel's *i-th* band value and $X_i$ is the *i-th* band value of selected pixel) as the criterion.
2. Prepare packaged versions - installer or a portable ZIP.
