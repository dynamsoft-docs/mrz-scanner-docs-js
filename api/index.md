---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - API Reference Index
keywords: Documentation, MRZ Scanner JavaScript Edition, API Index
description: MRZ Scanner JavaScript Edition Documentation API Reference Index
---

# API Reference Index

The MRZ Scanner JavaScript Edition exposes one main class — **MRZScanner** — together with a focused set of configuration and result interfaces. The single entry point [**`MRZScanner`**](mrz-scanner.md#mrzscanner) drives the entire scanning workflow; its configuration is layered through [**`MRZScannerConfig`**](mrz-scanner.md#mrzscannerconfig) and the nested [**`MRZScannerViewConfig`**](mrz-scanner.md#mrzscannerviewconfig). Scan results are returned as [**`MRZResult`**](mrz-scanner.md#mrzresult) objects carrying parsed [**`MRZData`**](mrz-scanner.md#mrzdata) and any opted-in [**`MRZImage`**](mrz-scanner.md#mrzimage) crops.

Read through the [**full API reference**](mrz-scanner.md), or use the summarized lists below to jump directly to a specific class, interface, or enum.

## Classes

1. [MRZScanner](mrz-scanner.md#mrzscanner) - The main class of the MRZ Scanner. Constructed with an [`MRZScannerConfig`](mrz-scanner.md#mrzscannerconfig) and used to launch the scanning workflow via [`launch()`](mrz-scanner.md#launch).

2. [MRZScannerView](mrz-scanner.md#mrzscannerview) - The live-camera scanner UI. Exported by the SDK but constructed and managed internally by `MRZScanner`; application code does not instantiate it directly.

## Configuration Interfaces

1. [MRZScannerConfig](mrz-scanner.md#mrzscannerconfig) - The top-level configuration interface for the `MRZScanner`. Configures the license, supported MRTD formats, the set of returned images, and the nested `MRZScannerView` configuration.

2. [MRZScannerViewConfig](mrz-scanner.md#mrzscannerviewconfig) - Configures the live-camera `MRZScannerView` UI: visible elements, the multi-side flip timeout, the load-image flow, and the four UI customization sub-configs listed below.

3. [ToolbarButtonsConfig](mrz-scanner.md#toolbarbuttonsconfig) - Per-button overrides for the seven `MRZScannerView` toolbar buttons.

4. [ToolbarButton](mrz-scanner.md#toolbarbutton) - The full shape of a single toolbar button.

5. [ToolbarButtonConfig](mrz-scanner.md#toolbarbuttonconfig) - The override shape applied to a single toolbar button.

6. [FormatSelectorConfig](mrz-scanner.md#formatselectorconfig) - Override labels for the four format selector buttons.

7. [MessagesConfig](mrz-scanner.md#messagesconfig) - Override every on-screen message displayed in the scanner view.

8. [ThemeConfig](mrz-scanner.md#themeconfig) - Override the scanner overlay's CSS color, typography, and spacing tokens.

## Result Interfaces

1. [MRZResult](mrz-scanner.md#mrzresult) - The full result returned by `launch()`. Carries the status, the parsed `MRZData`, and three getter methods exposing the captured images.

2. [MRZImage](mrz-scanner.md#mrzimage) - The image type returned by the `MRZResult` image getters. Extends `DSImageData` with `toCanvas()` and `toBlob()` helpers.

3. [MRZData](mrz-scanner.md#mrzdata) - The parsed MRZ fields (names, document number, dates, nationality, etc.).

4. [MRZDate](mrz-scanner.md#mrzdate) - The shape of date fields on `MRZData`.

5. [ResultStatus](mrz-scanner.md#resultstatus) - A standalone helper type pairing an `EnumResultStatus` code with an optional message.

## Enumerations

All enumerations live in [**`enums-mrz-scanner.md`**](enums-mrz-scanner.md). Summarized list:

1. [EnumMRZDocumentType](enums-mrz-scanner.md#enummrzdocumenttype) - The MRTD formats the scanner can be restricted to.

2. [EnumDocumentSide](enums-mrz-scanner.md#enumdocumentside) - Identifies the MRZ-bearing side and the opposite side of a document when retrieving images.

3. [EnumResultStatus](enums-mrz-scanner.md#enumresultstatus) - The status code carried by `MRZResult.status`.

4. [EnumMRZData](enums-mrz-scanner.md#enummrzdata) - The set of field keys on the parsed `MRZData` object.

## Helpers

1. [displayMRZDate](mrz-scanner.md#displaymrzdate) - Formats an `MRZDate` as a `YYYY-MM-DD` string.

2. [MRZDataLabel](mrz-scanner.md#mrzdatalabel) - A map of `EnumMRZData` keys to human-readable labels.
