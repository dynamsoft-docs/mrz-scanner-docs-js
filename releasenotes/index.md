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

## 3.0.2 (09/10/2025)

### Fixes

- Updated the underlying Capture Vision bundle to `3.0.6001`.
- Improved recognition speed by fixing an issue where a WASM compilation parameter caused performance degradation.
- Resolving security vulnerabilities regarding some of the dev dependencies of the framework samples.

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
