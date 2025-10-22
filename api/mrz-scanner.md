---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition - API Reference
keywords: Documentation, MRZ Scanner JavaScript Edition, API, APIs
breadcrumbText: API References
description: Dynamsoft MRZ Scanner JavaScript Edition - API Reference
---

# MRZ Scanner JavaScript Edition API Reference

The `MRZScanner` class is responsible for the main scanning process, including MRZ recognition, text parsing, and result display.

## Constructor

### MRZScanner

#### Syntax

```ts
new MRZScanner(config: MRZScannerConfig)
```

#### Parameters

- `config`: Assigns the MRZ Scanner license and configures the different settings of the MRZ Scanner, including the container, supported MRTD formats, and more. This config object is of type [*MRZScannerConfig*](#mrzscannerconfig).

#### Example

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
    license: "YOUR_LICENSE_KEY_HERE",
    scannerViewConfig: {
        // For custom scanning configuration, MRZScannerViewConfig goes in here
    },
    resultViewConfig: {
        // For custom result configuration, MRZResultViewConfig goes in here
    }
});
```

## Methods

### launch()

Starts the **MRZ scanning workflow**. If the method is run without a file input, the MRZScannerView UI will pop up and allow the user to scan MRZs using their camera. Alternatively, if the method is run with a file input parameter, the MRZ Scanner will read the MRZ from the file and display the result in a `MRZResultView`.

#### Syntax

```ts
launch(): Promise<MRZResult>

launch(fileToProcess): Promise<MRZResult>
```

#### Returns

A `Promise` resolving to a `MRZResult` object.

#### Example

```ts
(async () => {
    // Launch the scanner and wait for the result
    try {
        const result = await mrzScanner.launch();
        console.log(result); // print the MRZResult to the console
    } catch (error){
        console.error(error);
    }
})();
```

```ts
(async () => {
    // Launch the scanner and wait for the result
    try {
        const result = await mrzScanner.launch(fileToProcess);
        console.log(result); // print the MRZResult to the console
    } catch (error){
        console.error("Error processing file:", error);
    }
})();
```

### dispose()

Cleans up the MRZ Scanner resources and hides UI components.

#### Syntax

```ts
dispose(): void
```

#### Example

```ts
mrzScanner.dispose();
console.log("Scanner resources released.");
```

## Configuration Interfaces

### MRZScannerConfig

The **MRZScannerConfig** is responsible for assigning the MRZ Scanner license, configuring the MRTD formats, and setting the MRZScannerViewConfig and MRZResultViewConfig. Please note that the only thing that is **required** to be defined is the **license**. A MRZ Scanner instance **must** be initialized with a MRZScannerConfig object.

To get the full picture on how to use *MRZScannerConfig*, please visit the [Customizing MRZ Scanner - MRZScannerConfig]({{ site.guides }}mrz-scanner-customization.html#mrzscannerconfig-overview) page.

#### Syntax

```ts
interface MRZScannerConfig {
  license?: string; // license is absolutely required to be set at initialization
  container?: HTMLElement | string; //

  // Resource/Template specific configuration
  templateFilePath?: string;
  utilizedTemplateNames?: UtilizedTemplateNames;
  engineResourcePaths?: EngineResourcePaths;

  // Views Configuration
  scannerViewConfig?: Omit<MRZScannerViewConfig, "templateFilePath" | "utilizedTemplateNames">;
  resultViewConfig?: MRZResultViewConfig;

  mrzFormatType?: Array<EnumMRZDocumentType>;
  showResultView?: boolean;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `license`               | `string`                       | The license key for using the `MRZScanner`.                |
| `container`             | `HTMLElement \ string`        | The container element or selector for the `MRZScanner` UI. |
| `templateFilePath`      | `string`                       | The file path to the custom JSON template used to customize the scanning process.  |
| `utilizedTemplateNames` | `UtilizedTemplateNames`        | Specifies MRZ scanning templates.              |
| `engineResourcePaths`   | `EngineResourcePaths`          | Paths to the necessary resources for the MRZ scanning engine.  |
| `scannerViewConfig`     | [`MRZScannerViewConfig`](#mrzscannerviewconfig)         | Configuration settings for the MRZ scanner view.                    |
| `resultViewConfig`      | [`MRZResultViewConfig`](#mrzresultviewconfig)          | Configuration settings for the MRZ result view.                     |
| `mrzFormatType`         | [`EnumMRZDocumentType`]({{ site.api }}enums-mrz-scanner.html#enummrzdocumenttype)          | Specifies the MRTD formats that the application will support.  |
| `showResultView`        | `boolean`                      | Determines whether the final result view (MRZResultView) will be shown or not. |

#### Example

```ts
const mrzConfig = {
    license: "YOUR_LICENSE_KEY_HERE",
    mrzFormatType: ["passport", "td1"], // set the MRTD formats to just passport and ID (TD1)
    showResultView: false, // hide the final MRZResultView and go back to landing page once the MRZ result is in
    // engineResourcePaths typically is only assigned when using a framework like React/Angular/Vue where the resources are not in the same location as the script reference
    engineResourcePaths: {
        std: "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-std/dist/",
        dip: "https://cdn.jsdelivr.net/npm/dynamsoft-image-processing/dist/",
        core: "https://cdn.jsdelivr.net/npm/dynamsoft-core/dist/",
        license: "https://cdn.jsdelivr.net/npm/dynamsoft-license/dist/",
        cvr: "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-router/dist/",
        dlr: "https://cdn.jsdelivr.net/npm/dynamsoft-label-recognizer/dist/",
        dcp: "https://cdn.jsdelivr.net/npm/dynamsoft-code-parser/dist/",
    },
    scannerViewConfig: {
        // the MRZScannerViewConfig goes in here - see MRZScannerViewConfig section
    },
    resultViewConfig: {
        // the MRZResultViewConfig goes in here - see MRZResultViewConfig section
    }
};

// Initialize the Dynamsoft MRZ Scanner with the above MRZScannerConfig object
const mrzScanner = new Dynamsoft.MRZScanner(mrzConfig);
```

### MRZScannerViewConfig

The MRZScannerViewConfig is used to configure the UI elements of the **MRZScannerView**. If the MRZScannerViewConfig is not assigned, then the library will use the default MRZScannerView. For the full details of the properties of the MRZScannerViewConfig, please read through the [Customizing the MRZ Scanner - MRZScannerViewConfig]({{ site.guides }}mrz-scanner-customization.html#mrzscannerviewconfig-overview) page.

#### Syntax

```ts
interface MRZScannerViewConfig {
  uiPath?: string;
  container?: HTMLElement | string;

  showScanGuide?: boolean;
  showUploadImage?: boolean;
  showFormatSelector?: boolean;
  showSoundToggle?: boolean;
  showPoweredByDynamsoft?: boolean;

  enableMultiFrameCrossFilter?: boolean; // false by default

  uploadAcceptedTypes?: string;
  uploadFileConverter?: (file: File) => Promise<Blob>;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `uiPath`  | `string`                 | Path to the custom Camera Enhancer UI (`.html` template file) for the scanner view.               |
| `container`             | `HTMLElement \ string`  | The container element or selector for the `MRZScannerView` UI. |
| `showScanGuide`         | `boolean`                | Determines whether the scan guide frame will be displayed or not.  |
| `showUploadImage`       | `boolean`                | Determines the visibility of the "load image" icon to allow the user to select a static image from their local photo library.              |
| `showFormatSelector`    | `boolean`                | Determines the visibility of the format selector box that allows the user to restrict the MRTD format(s) that are being read.  |
| `showSoundToggle`       | `boolean`                | Determines the visibility of the "sound" icon that allows the user to play a beep sound once the MRZ is recognized.        |
| `showPoweredByDynamsoft`       | `boolean`         | Determines the visibility of the "Powered By Dynamsoft" imessage at the bottom of the scanner view.        |
| `enableMultiFrameCrossFilter`      | `boolean`     | Enables or disables the MultiFrameResultCrossFilter that can improve the accuracy of the MRZ result, but possibly at the cost of speed.   |
| `uploadAcceptedTypes`      | `string`     | Configures which image and file format(s) the library will accept if the user chooses to decode static images. |
| `uploadFileConverter`      | `function`     | Converts non-image files (e.g. PDF) to blobs so that they can be read by the MRZ Scanner.  |

#### Example

```ts
const mrzScanViewConfig = {
    showScanGuide: true, // hides the scan guide frame; true by default
    showUploadImage: true, // hides the load image icon that shows up in the toolbar at the top of the view; true by default
    showFormatSelector: true, // hides the format selector box if more than two MRZ types are assigned; true by default
    showSoundToggle: false, // hides the sound icon that allows the user to control whether a beep is played once an MRZ is recognized; true by default
    showPoweredByDynamsoft: false; // hides the "Powered By Dynamsoft" message that appears on the scanner UI; true by default
    enableMultiFrameCrossFilter: true, // turning the filter off could improve the speed but at the cost of result accuracy; true by default

    uploadAcceptedTypes: "image/*,application/pdf", // allows the user to upload static images and PDFs to be read by the MRZ Scanner - default is "image/*"
    uploadFileConverter: async (file) => {
        if (file.type === "application/pdf") {
            // Example PDF to image conversion
            const pdfData = await convertPDFToImage(file);
            return pdfData;
        }

        // For other non-image types, you can add more conversion logic
        // If it's not a supported type, throw an error
        throw new Error("Unsupported file type");
    },
};

const mrzConfig = {
    license: "YOUR_LICENSE_KEY_HERE",
    scannerViewConfig: mrzScanViewConfig
};
```

### MRZResultViewConfig

The MRZResultViewConfig is used to configure the UI elements of the **MRZResultView**. If the MRZResultViewConfig is not assigned, then the library will use the default MRZResultView. For the full details of the properties of the MRZResultViewConfig, please read through the [Customizing the MRZ Scanner - MRZResultViewConfig]({{ site.guides }}mrz-scanner-customization.html#mrzresultviewconfig-overview) page.

#### Syntax

```ts
interface MRZResultViewConfig {
  container?: HTMLElement | string;
  toolbarButtonsConfig?: MRZResultViewToolbarButtonsConfig;
  showOriginalImage?: boolean;
  allowResultEditing?: boolean; // New option to control if result fields can be edited
  showMRZText?: boolean;
  emptyResultMessage?: string;

  onDone?: (result: MRZResult) => Promise<void>;
  onCancel?: (result: MRZResult) => Promise<void>;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `container`             | `HTMLElement \ string`  | The container element or selector for the `MRZResultView` UI. |
| `toolbarButtonsConfig`  | `MRZResultViewToolbarButtonsConfig`  | Configures the main bottom toolbar of the result view.  |
| `showOriginalImage`     | `boolean`                | Determines whether the cropped image of the MRZ document will be displayed in the result view or not.              |
| `allowResultEditing`    | `boolean`                | Enables/disables the ability to edit the MRZ info after it is scanned.  |
| `showMRZText`    | `boolean`                | Shows/hides the raw MRZ text as one of the fields in the result view.  |
| `emptyResultMessage`    | `string`                | Sets the message to be displayed in the result view when no MRZ is detected.  |
| `onDone`      | `Promise<void>`     | Defines the action(s) to take once the user clicks the "Done" button in the result view.      |
| `onCancel`      | `Promise<void>`     | Defines the action(s) to take once the user clicks the "Cancel" button in the result view when the MRZ scanner is launched with a static file.      |

#### Example

```ts
const mrzResultViewConfig = {
    showOriginalImage: false, // Hides the cropped image of the MRZ document in the result view; true by default
    allowResultEditing: true, // Allows the user to manually edit the text of the parsed result fields; false by default
    showMRZText: false, // Hides the raw MRZ text as a field in the result view; true by default
    emptyResultMessage: "No MRZ is detected. Please try again.", // Change the message if there is no MRZ is detected. The default message is "The necessary information couldn't be detected. Please try again."
    toolbarButtonsConfig: {
        retake: {
            label: "Re-scan", // Changes the text label of the retake button to "Re-scan"; string is "Re-take" by default
            isHidden: true // Hides the retake button; false by default
        },
        done: {
            label: "Return Home", // Changes the text label of the done button to "Return Home"; string is "Done" by default
            isHidden: false // Hides the done button; false by default
        },
        cancel: {
            label: "Try Again", // Changes the text label of the cancel button; string is "Cancel" by default
            isHidden: false // Hides the cancel button; false by default.
        }
    },
    /* onDone is a callback that allows you to define the action(s) to take once the user clicks the Done button and the scanner is closed. By default, nothing happens and the app goes back to the landing page. */
    onDone: (result) => {
        console.log(result.status.message);
        console.log(result.data.firstName);
    },
    /* onCancel is a callback that allows you to define the action(s) that will happen once the user clicks the Cancel button which only shows up if the MRZ scanner is launched with a static file rather than with the camera view. By default, nothing happens and the app goes back to the landing page */
    onCancel: (result) => {
        /* implement the code to redirect the user to a new page should you not want them to go back to the landing page since that is the default behaviour */
    }
};

const mrzConfig = {
    license: "YOUR_LICENSE_KEY_HERE",
    resultViewConfig: mrzResultViewConfig
};
```

### MRZResultViewToolbarButtonsConfig

The `MRZResultViewToolbarButtonsConfig` is used to configure the buttons of the toolbar in the **MRZResultView**.

#### Syntax

```ts
interface MRZResultViewToolbarButtonsConfig {
  retake?: ToolbarButtonConfig;
  done?: ToolbarButtonConfig;
  cancel?: ToolbarButtonConfig;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `retake`    | [`ToolbarButtonConfig`](#toolbarbuttonconfig)  | Configuration for the re-scan button of the toolbar.   |
| `done`      | [`ToolbarButtonConfig`](#toolbarbuttonconfig)  | Configuration for the done button of the toolbar.  |
| `cancel`      | [`ToolbarButtonConfig`](#toolbarbuttonconfig)  | Configuration for the cancel button of the toolbar (which only shows up if the MRZ Scanner is launched with a static file instead of the standard camera UI).  |

#### Example

```ts
const mrzButtonConfig = {
    retake: {
        label: "Re-scan",
        isHidden: false
    },
    done: {
        label: "Return Home",
        isHidden: false
    },
    cancel: {
        label: "Try Again",
        isHidden: false
    }
};

const mrzResultViewConfig = {
    toolbarButtonsConfig: mrzButtonConfig,
};
```

### ToolbarButtonConfig

The interface used to configure the properties of the toolbar button. This interface is typically used in conjunction with the `MRZResultViewToolbarButtonsConfig`.

#### Syntax

```ts
export interface ToolbarButtonConfig {
  icon: string;
  label: string;
  className?: string;
  isHidden?: boolean;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `icon`       | `string`  | The path to a custom icon (png/svg) for the button.  |
| `label`      | `string`  | The text label of the button.  |
| `className`  | `string`  | Assigns a custom class to the button (usually to apply custom styling).  |
| `isHidden`   | `boolean` | Hides/Shows the button in the toolbar.  |

#### Example

See the [example of `MRZResultViewToolbarButtonsConfig`](#example-6).

## Result Interfaces

### MRZResult

The MRZResult is the most basic interface that is used to represent the full MRZ result object. It comes with a result status, the original cropped image result, and the parsed MRZ info as a [`MRZData`](#mrzdata) object.

#### Syntax

```ts
export interface MRZResult {
  status: ResultStatus;
  originalImageResult?: DSImageData;
  data?: MRZData;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `status`               | [`ResultStatus`](#resultstatus)  | The status of the MRZ result, which can be successful, cancelled (if the user closes the scanner), or failed (indicating that something has gone wrong during the scanning process).  |
| `originalImageResult`  | [`DSImageData`]({{ site.dcvb_js_api }}core/basic-structures/ds-image-data.html)  | A `DSImageData` object that represents the cropped image of the MRZ document.  |
| `data`                 | [`MRZData`](#mrzdata)                | Represents the parsed MRZ info.         |

#### Example

```ts
(async () => {
    // Launch the scanner and wait for the result
    const result = await mrzScanner.launch();
    console.log(result.status.message); // prints the result status message to the console
    console.log(result.status.code); // prints the result status code to the console
    console.log(result.data); // prints the entire MRZData object of the result to the console
})();
```

### MRZData

MRZData is used to represent the parsed fields of the MRZ result. These fields include the first name, last name, sex, document number, and more.

#### Syntax

```ts
export interface MRZData {
  [EnumMRZData.InvalidFields]?: EnumMRZData[]; //invalidFields
  [EnumMRZData.DocumentType]: string; // documentType
  [EnumMRZData.DocumentNumber]: string; // documentNumber
  [EnumMRZData.MRZText]: string; // mrzText
  [EnumMRZData.FirstName]: string; // firstName
  [EnumMRZData.LastName]: string; // lastName
  [EnumMRZData.Age]: number; // age
  [EnumMRZData.Sex]: string; // sex
  [EnumMRZData.IssuingState]: string; // issuingState
  [EnumMRZData.IssuingStateRaw]: string; //issuingStateRaw
  [EnumMRZData.Nationality]: string; // nationality
  [EnumMRZData.NationalityRaw]: string; //nationalityRaw
  [EnumMRZData.DateOfBirth]: MRZDate; // dateOfBirth
  [EnumMRZData.DateOfExpiry]: MRZDate; // dateOfExpiry
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `documentType`    | `string`  | The type of MRTD that the MRZ document is. |
| `documentNumber`  | `string`  | The MRZ document number.  |
| `mrzText`         | `string`  | The raw unparsed text of the MRZ.              |
| `firstName`       | `string`  | The first name of the MRZ document holder.  |
| `lastName`        | `string`  | The last name of the MRZ document holder.      |
| `age`             | `string`  | The age of the MRZ document holder.      |
| `sex`             | `string`  | The sex of the MRZ document holder.      |
| `issuingState`    | `string`  | The issuing state (represented as the full name of the country/region) of the MRZ document.     |
| `issuingStateRaw`    | `string`  | The raw text from the MRZ string of the issuing state of the MRZ document.     |
| `nationality`     | `string`  | The nationality (represented as the full name of the country/region) of the MRZ document holder.      |
| `nationalityRaw`     | `string`  | The raw text from the MRZ string representing the nationality of the document holder.      |
| `dateOfBirth`     | [`MRZDate`](#mrzdate) | The date of birth of the MRZ document holder.      |
| `dateOfExpiry`    | [`MRZDate`](#mrzdate) | The date of expiry of the MRZ document.      |

#### Example

```ts
(async () => {
    // Launch the scanner and wait for the result
    const result = await mrzScanner.launch();
    console.log(result.data.firstName); // prints the first name
    console.log(result.data.lastName); // prints the last name
    console.log(result.data.sex); // prints the sex
    console.log(result.data.nationality); // prints the nationality
    console.log(result.data.documentNumber); // prints the MRZ document number
})();
```

### MRZDate

MRZDate is used to represent the date fields of a `MRZData` object.

#### Syntax

```ts
interface MRZDate {
  year: number;
  month: number;
  day: number;
}
```

#### Properties

| Property                | Type                           | Description                                                     |
| ----------------------- | ------------------------------ | --------------------------------------------------------------- |
| `year`    | `number`  | The year of the date.   |
| `month`   | `number`  | The month of the date.  |
| `day`     | `number`  | The day of the date.    |

#### Example

```ts
(async () => {
    // Launch the scanner and wait for the result
    const result = await mrzScanner.launch();
    console.log(result.data.dateOfBirth.year); // prints the year of the date of birth
    console.log(result.data.dateOfBirth.month); // prints the month of the date of birth
    console.log(result.data.dateOfBirth.day); // prints the day of the date of birth
})();
```

### ResultStatus

ResultStatus is used to represent the status of the MRZ Result. This status can be **successful**, **cancelled** if the user closes the scanner, or **failed** if something went wrong during the scanning process. The *code* of the result status is a [`EnumResultStatus`]({{ site.api }}enums-mrz-scanner.md#enumresultstatus).

#### Syntax

```ts
type ResultStatus = {
  code: EnumResultStatus;
  message?: string;
};
```

#### Example

Please see the [example of MRZResult](#example-8).
