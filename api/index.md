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

The MRZ Scanner library comes with a number of classes and interfaces which we will dive into in the API documentation. There are three main classes: **MRZScanner**, **MRZScannerView**, **MRZResultView**, and **MRZResult**. In terms of interfaces, the main ones are: **MRZScannerConfig**, **MRZScannerViewConfig**, and **MRZResultViewConfig**.

Please read through the [**full API reference**](mrz-scanner.md), but you can find a summarized list of the classes, interfaces, and enums below.

## Classes

1. [MRZScanner](mrz-scanner.md#mrzscanner) - The main class of the MRZ Scanner, which is used to create and configure the MRZ Scanner instance.

2. MRZScannerView - Represents the main view of the MRZ Scanner where the scanning operation occurs.

3. MRZResultView - Displays the parsed MRZ result in human readable fields, along with a cropped image of the MRZ document.

## Interfaces

1. [MRZScannerConfig](mrz-scanner.md#mrzscannerconfig) - The main configuration class of the MRZScanner. It is used to assign the supported MRTD formats of your application as well as the configurations of the **MRZScannerView** and the **MRZResultView**.

2. [MRZScannerViewConfig](mrz-scanner.md#mrzscannerviewconfig) - Configures the different UI elements of the **MRZScannerView**.

3. [MRZResultViewConfig](mrz-scanner.md#mrzresultviewconfig) - Configures the different UI elements of the **MRZResultView**.

4. [MRZResultViewToolbarButtonsConfig](mrz-scanner.md#mrzresultviewtoolbarbuttonsconfig) - Configures the toolbar buttons of the **MRZResultView**.

5. [MRZResult](mrz-scanner.md#mrzresult) - Represents a typical MRZ result along with all of the parsed fields that come with it.

6. [MRZData](mrz-scanner.md#mrzdata) - Represents the parsed MRZ data that is part of the `MRZResult`.

7. [MRZDate](mrz-scanner.md#mrzdate) - Represents a date in the MRZ fields - which is usually used for date of birth and the date of expiry.

## Enumerations

All of the enumerations can be found [**here**](enums-mrz-scanner.md). Here is a summarized list of the available enumerations.

1. [EnumMRZDocumentType](enums-mrz-scanner.md#enummrzdocumenttype) - An enumeration to represent the different types of MRTD formats that the MRZScanner supports.

2. [EnumResultStatus](enums-mrz-scanner.md#enumresultstatus) - An enumeration to represent the status of a MRZ result.

3. [EnumMRZData](enums-mrz-scanner.md) - An enumeration to represent the different fields of the `MRZData` interface.
