---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - Release Notes
keywords: Documentation, MRZ Scanner JavaScript Edition, Release Notes
breadcrumbText: Release Notes
description: MRZ Scanner JavaScript Edition Documentation Release Notes
permalink: /releasenotes/index.html
---

# Release Notes

## 4.0.0 (05/22/2026)

A major release with a redesigned scanner UI, a new image-extraction pipeline, multi-side scanning, first-class visa support, and a leaner API surface that drops the bundled result view in favor of consumer-rendered results.

### Highlighted Features

- **Image extraction in scan results.** Opt in via `returnOriginalImage` / `returnDocumentImage` / `returnPortraitImage` on `MRZScannerConfig`, then read the deskewed crop, the full original frame, and the portrait crop via `MRZResult.getDocumentImage(side)`, `getOriginalImage(side)`, and `getPortraitImage()`. Each returns a `DSImageData` with `toCanvas()` / `toBlob()` helpers.
- **Multi-side scanning.** When the portrait sits on the opposite side of the MRZ (typical of TD1 / TD2 ID cards), the scanner prompts the user to flip the document and captures both sides in one session. The new `EnumDocumentSide` (`MRZ` / `Opposite`) selects which side to retrieve. Tunable via `flipDocumentTimeout` (default `3000`ms).
- **First-class visa types.** Visas are no longer conflated with Passport / TD2. `EnumMRZDocumentType` gains `MRVA` (TD3-sized) and `MRVB` (TD2-sized), with matching values in `EnumMRZScanMode`.
- **Redesigned `MRZScannerView`.** New SVG guide frames, animated scan spinner, and refreshed toolbar. Four new `scannerViewConfig` interfaces give full customization: `toolbarButtonsConfig` (per-button icon / label / className / visibility), `formatSelectorConfig` (localize format buttons), `messagesConfig` (localize all on-screen text, including a `{seconds}` placeholder for the flip-document countdown), and `themeConfig` (color, typography, and spacing tokens).
- **URL input to `launch()`.** `launch()` now accepts a string URL alongside `Blob`, `DSImageData`, and HTML media elements; the library fetches and decodes the URL internally.
- **`OptionalData1` and `OptionalData2`** ICAO optional-data fields now exposed on `EnumMRZData`. `MRZData.documentType` is now typed as DCV's `EnumCodeType`.

### Breaking Changes

Migrating an existing v3.x integration? Work through the [Migration Guide (v3.x to v4.0)]({{ site.guides }}mrz-scanner-upgrade-guide.html), which covers each change below in detail with before/after code.

- **`MRZResultView` is removed**, along with `MRZResultViewConfig`, `MRZResultViewToolbarButtonsConfig`, `resultViewConfig`, and `showResultView`. Consumers render the result themselves — see the [User Guide]({{ site.guides }}mrz-scanner.html) for a worked example.
- **`MRZResult.originalImageResult` and `MRZResult.imageData` are removed**, replaced by `getOriginalImage(side)`. The MWC-specific `_imageData` internal is also gone.
- **API renames:** `showUploadImage` → `showLoadImageButton`, `uploadAcceptedTypes` → `loadImageAcceptedTypes`, `uploadFileConverter` → `loadImageFileConverter`, `showScanGuide` → `enableScanRegion`, `EnumMRZScanMode.Passport` → `EnumMRZScanMode.TD3`.
- **`EnumMRZDocumentType` string values changed** to disambiguate visas: `Passport`'s value is now `"td3_passport"` (was `"passport"`), `TD1` is `"td1_id"` (was `"td1"`), `TD2` is `"td2_id"` (was `"td2"`). Code using the enum references is unaffected; raw-string comparisons must be updated.
- **`MRZResult` shape changed.** Now an interface with image-getter methods. The `status` field is optional and uses `EnumResultStatus` directly (was a `ResultStatus` object with `code` / `message`).
- **DCV namespace flattened.** DCV exports are now reachable directly under `Dynamsoft.*` instead of the previous `Dynamsoft.Dynamsoft.*` nesting.

### Fixes & Improvements

- Upgraded the underlying DCV to **3.4.2001** under new package names: `dynamsoft-capture-vision-bundle` and `dynamsoft-capture-vision-data`.
- New scan-quality controls: `enableMultiFrameCrossFilter` (default `true`) for DCV multi-frame cross-verification, and `enableScanRegion` (default `true`) for the rectangular guide frame.
- Engine resource paths now resolve relative to the script's own URL, so consumers no longer hard-code CDN paths.
- `UtilizedTemplateNames` accepts either a string or a `TemplatePair` (`{ full, mrzOnly }`) per scan mode, enabling separate full-frame and MRZ-only templates.
- Reduced repeated camera permission prompts on Firefox Android.
- Removed spurious TD3-visa matches from passport scanning.
- Prevented portrait-image clipping on capture via asymmetric margins.
- Format selector and scan results now respect `mrzFormatType`, only surfacing the categories the consumer opted into.
- Visual guide-frame visibility properly respects `enableScanRegion`.
- The configured `cameraAccessDenied` message is now displayed on permission denial.
- Bundle pruned to ESM + CJS + a single IIFE; template files renamed from `.html` to `.xml` (`mrz-scanner.ui.xml`, `mrz-scanner.template.json`); fixed nested `dist/` artifact in the published package.

## 3.1.0 (01/13/2026)

### Highlighted Features

- Neural MRZ Localization – The new [MRZLocalization](https://www.dynamsoft.com/capture-vision/docs/core/parameters/reference/label-recognizer-task-settings/localization-modes.html#modelnamearray) model improves region detection accuracy and delivers up to 42.7% faster processing for MRZ-based document workflows.
- Configurable Localization Control – The new [LocalizationModes](https://www.dynamsoft.com/capture-vision/docs/core/parameters/reference/label-recognizer-task-settings/localization-modes.html) parameter allows configuration for text line detection.

### Fixes & Improvements

- Updated third-party libraries to incorporate the latest security fixes and maintenance updates.

## 3.0.4 (10/20/2025)

### Fixes

- Fixed a bug with the `mrzText` field of the `MRZData` interface.

## 3.0.3 (09/17/2025)

### Fixes

- Fixed the `launch()` method so that the `imageOrFile` parameter is optional (for TypeScript implementation).
- Altered the React Hooks sample to allow it to run in `Strict Mode`.

## 3.0.2 (09/11/2025)

### Fixes

- Updated the underlying Capture Vision bundle to `3.0.6001`.
- Improved recognition speed by fixing an issue where a WASM compilation parameter caused performance degradation.
- Strengthened and improved the framework samples (e.g. Angular) with security updates.

## 3.0.1 (08/06/2025)

### Fixes

- Fixed the file input process error when decoding static file input when `showResultView` is set to `false`.
- Resolved an issue with the `launch()` method throwing an error when there's a different error format thrown.

## 3.0.0 (06/17/2025)

### Highlighted Features

- Updated the underlying Capture Vision bundle to `3.0.3001` for major improvements in reading accuracy and speed.
- Optimized the algorithm to achieve a **30% increase in read rate** as well as a **15% increase in accuracy**.
- Added support for `TD2` and `TD3` Visa.
- Added a `emptyResultMessage` property to the `ResultViewConfig` interface in order to change the string message that is displayed when no result is found.

### Fixes

- Fixed the issue where the camera select icon cuts off on browsers in iOS.
- Optimized the resource loading process of the library. 
- Replaced the re-scan button of the result view with a cancel button when the MRZ scanner is launched with a static file.

## 2.1.0 (05/16/2025)

### Highlighted Features

- **[UI]** Redesigned the **`MRZScannerView`** (the main camera view) to have updated icons and better alignment and spacing.
- Changed the default camera resolution when the camera is opened from **1080p** to **2K** (if the camera supports it).
- Added a new property to the **`MRZScannerViewConfig`** interface, **`showPoweredByDynamsoft`**, which controls the visibility of the `Powered By Dynamsoft` message that is part of the **`MRZScannerView`** UI.
- Introduced the ability for the MRZ Scanner to read MRZs from static images and PDFs without the need to use the **`MRZScannerView`** UI and the Load Image button.
- Added two new properties to the **`MRZScannerViewConfig`** interface, **`uploadAcceptedTypes`** and **`uploadFileConverter`**, which convert static images and PDFs to blobs which can then be read by the MRZ Scanner.
- Redeveloped the `launch()` method so that it can take a static image or file as input. To learn how to use that, please refer to [Setting up the MRZ Reader for Static Images]({{ site.guides }}mrz-scanner-static-image.html)
- Integrated Dynamsoft's [Mobile Web Capture](https://www.dynamsoft.com/mobile-web-capture/docs/introduction/) with the MRZ Scanner (JavaScript Edition) to allow the user to edit the scanned MRZ image like a document.
- Added `NationalityRaw` and `IssuingStateRaw` to the `MRZData` interface that represent the raw values of these fields

### Fixes

- Fixed parsing of German IDs returning `D<<` instead of `D`.
- `engineResourcePaths` is now set before `initLicense` (internally) to prevent a bug when the user wants to implement a custom `engineResourcePaths`.
- Update the trial license banner link to point to https://www.dynamsoft.com/customer/license/trialLicense?product=mrz&deploymenttype=web

## 2.0.0 (03/13/2025)

The **MRZ Scanner JavaScript Edition** has been redesigned and redeveloped to now include a **ready-to-use, fully developed UI** to ease the development process while providing even better functionality.

### Highlighted Features

- Automatic detection and parsing of MRZs in passports and IDs
- Support for the following MRTD formats: TD3 (Passport), TD2 (ID), and TD1 (ID)
- Ready-to-use UI to simplify the development process
- Supports an interactive video scenario (capturing via video) as well as static images (jpg/png)
- Modular, view-based design for easy maintenance and customization

### Views

MRZ Scanner JavaScript Edition is organized into configurable UI views. Below is a quick overview of the two main views:

> [!TIP]
> Learn more about these views and how to configure them in the [User Guide]({{ site.guides }}mrz-scanner.html) and the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).

#### MRZ Scanner View

- Fully configurable camera view
- Resolution/camera select dropdown to allow for quick improvements
- Newly developed scan guide frame to enable easy capture of the MRZ document
- Green highlight overlay over the MRZ area once recognized
- Flash/Torch support
- Allows the user to load in an image from the photo library

#### MRZ Result View

- Scrollable field view to display the parsed MRZ info
- Displays the original image of the MRZ document
- Allows the user to edit almost all of the parsed fields
- Two buttons to allow the user to re-scan or move on
