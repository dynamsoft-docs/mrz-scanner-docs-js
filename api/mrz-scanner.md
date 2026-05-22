---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: MrzScanner API Reference – Dynamsoft JS Edition
keywords: Documentation, MRZ Scanner JavaScript Edition, API, APIs
breadcrumbText: API References
description: Dynamsoft MRZ Scanner JavaScript Edition - API Reference
---

# MRZ Scanner JavaScript Edition API Reference

The `MRZScanner` class is responsible for the main scanning process, including MRZ recognition, text parsing, and the production of cropped document and portrait images for your application to render.

## Quick Links

**Classes**

- [MRZScanner](#mrzscanner)
- [MRZScannerView](#mrzscannerview)

**Methods**

- [launch()](#launch)
- [dispose()](#dispose)

**Configuration Interfaces**

- [MRZScannerConfig](#mrzscannerconfig)
- [MRZScannerViewConfig](#mrzscannerviewconfig)
- [ToolbarButtonsConfig](#toolbarbuttonsconfig)
- [ToolbarButton](#toolbarbutton)
- [ToolbarButtonConfig](#toolbarbuttonconfig)
- [FormatSelectorConfig](#formatselectorconfig)
- [MessagesConfig](#messagesconfig)
- [ThemeConfig](#themeconfig)

**Result Interfaces**

- [MRZResult](#mrzresult)
- [MRZImage](#mrzimage)
- [MRZData](#mrzdata)
- [MRZDate](#mrzdate)
- [ResultStatus](#resultstatus)

**Helpers**

- [displayMRZDate](#displaymrzdate)
- [MRZDataLabel](#mrzdatalabel)

## Classes

### MRZScanner

#### Syntax

```ts
new MRZScanner(config: MRZScannerConfig)
```

#### Parameters

- `config`: Assigns the MRZ Scanner license and configures the different settings of the MRZ Scanner, including the container, supported MRTD formats, returned images, and the scanner view UI. This config object is of type [*MRZScannerConfig*](#mrzscannerconfig).

#### Example

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
    license: "YOUR_LICENSE_KEY_HERE",
    returnOriginalImage: true,
    returnDocumentImage: true,
    returnPortraitImage: true,
    scannerViewConfig: {
        // For custom scanning configuration, MRZScannerViewConfig goes in here
    }
});
```

### MRZScannerView

`MRZScannerView` is a public export of the SDK, but application code does not instantiate it directly — it is constructed and managed internally by [`MRZScanner`](#mrzscanner). All configuration of the live-camera scanner UI is exposed through [`MRZScannerViewConfig`](#mrzscannerviewconfig), nested under the `scannerViewConfig` field of [`MRZScannerConfig`](#mrzscannerconfig).

## Methods

### launch()

Starts the **MRZ scanning workflow**. If the method is run without an argument, the `MRZScannerView` UI will open and allow the user to scan an MRZ using their camera. Alternatively, if the method is called with a static image source (`Blob`, `File`, image URL, `DSImageData`, or an `HTMLImageElement`/`HTMLVideoElement`/`HTMLCanvasElement`), the MRZ Scanner will read the MRZ from that source directly without opening the camera view.

The promise resolves with an [`MRZResult`](#mrzresult). On successful scans, the result carries the parsed [`MRZData`](#mrzdata) and any opted-in images. On cancellation (the user closes the scanner) or failure, the result carries only the [`status`](#resultstatus) field — no `data` and no images. Your application is responsible for rendering the result; v4 does not include a built-in result view.

#### Syntax

```ts
launch(): Promise<MRZResult>

launch(imageSource: Blob | string | DSImageData | HTMLImageElement | HTMLVideoElement | HTMLCanvasElement): Promise<MRZResult>
```

#### Returns

A `Promise` resolving to a [`MRZResult`](#mrzresult) object.

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
    // Launch the scanner with a static file
    try {
        const result = await mrzScanner.launch(fileToProcess);
        console.log(result); // print the MRZResult to the console
    } catch (error){
        console.error("Error processing file:", error);
    }
})();
```

### dispose()

Tears down the MRZ Scanner — disposes camera handles, the Capture Vision router, and clears the DOM containers.

> [!NOTE]
> `launch()` automatically calls `dispose()` in its `finally` block on every resolution path (success, failure, or cancellation). You only need to call `dispose()` manually when you want to tear the scanner down **without** launching it again — for example, when unmounting a single-page-app component before any scan has been performed. Calling `dispose()` is idempotent.

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

The **MRZScannerConfig** is responsible for assigning the MRZ Scanner license, the supported MRTD formats, the returned image set, and the `MRZScannerView` configuration. The only required field is the **license** — an `MRZScanner` instance **must** be constructed with an `MRZScannerConfig` object.

To learn how to use *MRZScannerConfig* in practice, see [Customizing MRZ Scanner - MRZScannerConfig]({{ site.guides }}mrz-scanner-customization.html#mrzscannerconfig-overview).

#### Syntax

```ts
interface MRZScannerConfig {
  license?: string; // license is absolutely required to be set at initialization
  container?: HTMLElement | string;

  // Resource/Template specific configuration
  templateFilePath?: string;
  engineResourcePaths?: EngineResourcePaths;

  // Scanner View Configuration
  scannerViewConfig?: MRZScannerViewConfig;

  mrzFormatType?: Array<EnumMRZDocumentType>;

  // Returned-image flags (control what's attached to the MRZResult)
  returnOriginalImage?: boolean;
  returnDocumentImage?: boolean;
  returnPortraitImage?: boolean;
}
```

#### Properties

| Property                | Type                           | Default | Description                                                     |
| ----------------------- | ------------------------------ | ------- | --------------------------------------------------------------- |
| `license`               | `string`                       | —       | The license key for using the `MRZScanner`. **Required.** |
| `container`             | `HTMLElement \| string`        | —       | The container element or selector for the `MRZScanner` UI. When omitted, the scanner mounts a full-viewport `<div>` on `<body>`. |
| `templateFilePath`      | `string`                       | bundled | Path to a custom Capture Vision template JSON file. Contact the [Dynamsoft Technical Support Team](https://www.dynamsoft.com/company/contact/) for assistance with custom template work. |
| `engineResourcePaths`   | `EngineResourcePaths`          | —       | Paths to the DCV engine WASM and data files. Required when installing the SDK via npm. |
| `scannerViewConfig`     | [`MRZScannerViewConfig`](#mrzscannerviewconfig) | — | Configuration settings for the `MRZScannerView`. |
| `mrzFormatType`         | [`EnumMRZDocumentType`]({{ site.api }}enums-mrz-scanner.html#enummrzdocumenttype) array | all formats | Restricts recognition to a subset of MRTD formats. Strings are accepted (e.g. `"td3_passport"`) for readability. |
| `returnOriginalImage`   | `boolean`                      | `false` | When `true`, attaches the full unmodified frame on the result. Retrieved via `result.getOriginalImage(side)`. |
| `returnDocumentImage`   | `boolean`                      | `true`  | When `true`, attaches a deskewed crop of the document on the result. Retrieved via `result.getDocumentImage(side)`. |
| `returnPortraitImage`   | `boolean`                      | `true`  | When `true`, attaches a cropped portrait on the result and activates multi-side scanning when needed. Retrieved via `result.getPortraitImage()`. |

#### Example

```ts
const mrzConfig = {
    license: "YOUR_LICENSE_KEY_HERE",
    // Restrict the scanner to passports and TD1 ID cards.
    // Strings are used here instead of the EnumMRZDocumentType enum for readability.
    mrzFormatType: ["td3_passport", "td1_id"],

    // Return every image the SDK can produce so the result view has full data.
    returnOriginalImage: true,
    returnDocumentImage: true,
    returnPortraitImage: true,

    // engineResourcePaths is required when installing via npm — point at the DCV peers
    engineResourcePaths: {
        dcvBundle: "node_modules/dynamsoft-capture-vision-bundle/dist/",
        dcvData: "node_modules/dynamsoft-capture-vision-data/",
    },
    scannerViewConfig: {
        // the MRZScannerViewConfig goes in here - see MRZScannerViewConfig section
    }
};

// Initialize the Dynamsoft MRZ Scanner with the above MRZScannerConfig object
const mrzScanner = new Dynamsoft.MRZScanner(mrzConfig);
```

### MRZScannerViewConfig

The `MRZScannerViewConfig` configures the UI elements and behavior of the **`MRZScannerView`** — the live-camera view shown when `launch()` is called without a static image source. If `MRZScannerViewConfig` is not assigned, the scanner uses default values for every option. For practical usage details, see [Customizing the MRZ Scanner - MRZScannerViewConfig]({{ site.guides }}mrz-scanner-customization.html#mrzscannerviewconfig-overview).

#### Syntax

```ts
interface MRZScannerViewConfig {
  uiPath?: string;
  container?: HTMLElement | string;

  // Scanner UI visibility toggles
  enableScanRegion?: boolean;
  showLoadImageButton?: boolean;
  showFormatSelector?: boolean;
  showSoundToggle?: boolean;

  enableMultiFrameCrossFilter?: boolean;

  // Load-image flow
  loadImageAcceptedTypes?: string;
  loadImageFileConverter?: (file: File) => Promise<Blob>;

  // Multi-side scanning
  flipDocumentTimeout?: number;

  // UI customization
  toolbarButtonsConfig?: ToolbarButtonsConfig;
  formatSelectorConfig?: FormatSelectorConfig;
  messagesConfig?: MessagesConfig;
  themeConfig?: ThemeConfig;
}
```

#### Properties

| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `uiPath` | `string` | bundled | Path to a custom Camera Enhancer UI (`.html` template file) for the scanner view. |
| `container` | `HTMLElement \| string` | — | The container element or selector for the `MRZScannerView` UI. |
| `enableScanRegion` | `boolean` | `true` | When `true`, the scanner reads only the area inside the central guide frame. When `false`, the full camera frame is processed. |
| `showLoadImageButton` | `boolean` | `true` | Shows the gallery / load-image button in the toolbar. |
| `showFormatSelector` | `boolean` | `false` | Shows the format selector (passport / ID / visa / all) underneath the guide frame. Only renders when more than one format is enabled via `mrzFormatType`. |
| `showSoundToggle` | `boolean` | `true` | Shows the sound (beep on success) toggle button in the toolbar. |
| `enableMultiFrameCrossFilter` | `boolean` | `true` | Enables DCV's multi-frame cross-verification filter. Improves accuracy at the cost of a small latency increase. |
| `loadImageAcceptedTypes` | `string` | `"image/*"` | Value passed to the `<input type="file">` `accept` attribute when the user clicks the load-image button. |
| `loadImageFileConverter` | `(file: File) => Promise<Blob>` | — | Async callback that converts a non-image `File` (e.g. a PDF) into an image `Blob` before processing. Required to support PDF uploads. |
| `flipDocumentTimeout` | `number` (ms) | `3000` | Pause duration after the MRZ side is captured during multi-side scanning, giving the user time to flip the document before portrait-side scanning begins. |
| `toolbarButtonsConfig` | [`ToolbarButtonsConfig`](#toolbarbuttonsconfig) | — | Per-button overrides (icon, label, className, visibility) for the seven scanner toolbar buttons. |
| `formatSelectorConfig` | [`FormatSelectorConfig`](#formatselectorconfig) | — | Labels for the format selector buttons. |
| `messagesConfig` | [`MessagesConfig`](#messagesconfig) | bundled | Overrides for every on-screen message in the scanner view. |
| `themeConfig` | [`ThemeConfig`](#themeconfig) | bundled | Override the scanner overlay's CSS color, typography, and spacing tokens. |

#### Example

```ts
const mrzScanViewConfig = {
    enableScanRegion: true, // capture only the area inside the guide frame; true by default
    showLoadImageButton: true, // show the gallery button in the toolbar; true by default
    showFormatSelector: true, // show the passport/ID/visa/all selector; false by default
    showSoundToggle: false, // hide the sound toggle button; true by default
    enableMultiFrameCrossFilter: true, // multi-frame cross verification; true by default

    flipDocumentTimeout: 3000, // ms to wait for the user to flip the document during multi-side scanning

    loadImageAcceptedTypes: "image/*,application/pdf", // allow PDFs in addition to images
    loadImageFileConverter: async (file) => {
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

Four of the `MRZScannerViewConfig` fields take their own dedicated sub-config interfaces, each documented in its own section below:

- [`ToolbarButtonsConfig`](#toolbarbuttonsconfig) — per-button overrides for the seven scanner toolbar buttons.
- [`FormatSelectorConfig`](#formatselectorconfig) — labels for the four format selector buttons.
- [`MessagesConfig`](#messagesconfig) — every on-screen message displayed in the scanner view.
- [`ThemeConfig`](#themeconfig) — color, typography, and spacing tokens for the scanner overlay.

### ToolbarButtonsConfig

The `ToolbarButtonsConfig` configures the seven buttons in the `MRZScannerView` toolbar. Each key maps to a [`ToolbarButtonConfig`](#toolbarbuttonconfig), and any omitted button keeps its default rendering.

#### Syntax

```ts
interface ToolbarButtonsConfig {
  close?: ToolbarButtonConfig;
  loadImage?: ToolbarButtonConfig;
  cameraSwitch?: ToolbarButtonConfig;
  flash?: ToolbarButtonConfig;
  flashOff?: ToolbarButtonConfig;
  sound?: ToolbarButtonConfig;
  soundOff?: ToolbarButtonConfig;
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `close` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the close-scanner button. |
| `loadImage` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the gallery / load-image button. |
| `cameraSwitch` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the camera-switch button. |
| `flash` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the flash-on button. |
| `flashOff` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the flash-off button. |
| `sound` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the sound-on button. |
| `soundOff` | [`ToolbarButtonConfig`](#toolbarbuttonconfig) | Override for the sound-off button. |

#### Example

```ts
const toolbarButtonsConfig = {
    close: { label: "Cancel" },
    loadImage: { isHidden: true },
    cameraSwitch: { className: "my-custom-camera-switch" },
    flash: { icon: "<svg>...</svg>" },
    flashOff: { icon: "<svg>...</svg>" },
};

const mrzScanViewConfig = {
    toolbarButtonsConfig
};
```

### ToolbarButton

The full toolbar button type that every button in the `MRZScannerView` toolbar conforms to. [`ToolbarButtonConfig`](#toolbarbuttonconfig) is a partial of this with the internally-managed `id` field removed.

#### Syntax

```ts
interface ToolbarButton {
  id: string;
  icon?: string;
  label?: string;
  isHidden?: boolean;
  className?: string;
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `id` | `string` | Internal identifier for the button (managed by the SDK). |
| `icon` | `string` | Inline SVG markup or a URL to use as the button's icon. |
| `label` | `string` | Accessible label / tooltip text. |
| `isHidden` | `boolean` | When `true`, the button is not rendered. |
| `className` | `string` | Additional CSS class added to the button element. |

### ToolbarButtonConfig

The override shape applied to a single toolbar button — a `Partial` of [`ToolbarButton`](#toolbarbutton) with the internally-managed `id` field removed. Every field is optional; omitted fields fall back to the button's default rendering.

#### Syntax

```ts
type ToolbarButtonConfig = Partial<Omit<ToolbarButton, "id">>;
```

Equivalent to:

```ts
interface ToolbarButtonConfig {
  icon?: string;
  label?: string;
  className?: string;
  isHidden?: boolean;
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `icon` | `string` | Inline SVG markup or a URL to use as the button's icon. |
| `label` | `string` | Accessible label / tooltip text. |
| `className` | `string` | Additional CSS class added to the button element. |
| `isHidden` | `boolean` | When `true`, the button is not rendered. |

#### Example

See the [example of `ToolbarButtonsConfig`](#example-5).

### FormatSelectorConfig

Override the four format-selector button labels. Used in conjunction with `MRZScannerViewConfig.showFormatSelector: true`.

#### Syntax

```ts
interface FormatSelectorConfig {
  passportLabel?: string;
  idLabel?: string;
  visaLabel?: string;
  allLabel?: string;
}
```

#### Properties

| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `passportLabel` | `string` | `"Passport"` | Label for the passport button. Shown when a passport format is enabled. |
| `idLabel` | `string` | `"ID"` | Label for the ID button. Shown when a TD1 or TD2 ID format is enabled. |
| `visaLabel` | `string` | `"Visa"` | Label for the visa button. Shown when an MRVA or MRVB visa format is enabled. |
| `allLabel` | `string` | `"All"` | Label for the "All" button. Shown when more than one format category is enabled. |

#### Example

```ts
const formatSelectorConfig = {
    passportLabel: "Passport",
    idLabel: "ID Card",
    visaLabel: "Visa",
    allLabel: "All",
};

const mrzScanViewConfig = {
    showFormatSelector: true,
    formatSelectorConfig
};
```

### MessagesConfig

Override every on-screen message displayed in the `MRZScannerView`. Any key you omit keeps its default English value. The `flipDocumentCountdown` message supports a `{seconds}` placeholder that is replaced with the remaining seconds during the flip countdown.

#### Syntax

```ts
interface MessagesConfig {
  positionMRZ?: string;
  holdSteady?: string;
  scanSuccess?: string;
  flipDocument?: string;
  flipDocumentCountdown?: string;
  positionPortrait?: string;
  scanMRZFirst?: string;
  scanningPortrait?: string;
  portraitScanned?: string;
  bothSidesScanned?: string;
  skipPortraitLabel?: string;
  loadImageFailed?: string;
  cameraAccessDenied?: string;
}
```

#### Properties

| Property | Type | Default | When it appears |
| --- | --- | --- | --- |
| `positionMRZ` | `string` | `"Position MRZ within the frame"` | Idle MRZ-side scanning prompt. |
| `holdSteady` | `string` | `"Hold steady..."` | Briefly shown when a candidate MRZ is detected. |
| `scanSuccess` | `string` | `"MRZ scanned ✓"` | Confirmation after the MRZ side is captured. |
| `flipDocument` | `string` | `"Now scan the portrait side"` | Prompt to flip the document during multi-side scanning. |
| `flipDocumentCountdown` | `string` | `"Flip and scan the other side ({seconds}s)"` | Countdown shown during `flipDocumentTimeout`. Supports the `{seconds}` placeholder. |
| `positionPortrait` | `string` | `"Position portrait within the frame"` | Idle portrait-side scanning prompt. |
| `scanMRZFirst` | `string` | `"Scan the MRZ side first"` | Shown if a portrait is detected before the MRZ side is captured. |
| `scanningPortrait` | `string` | `"Scanning portrait..."` | Active portrait-side scanning. |
| `portraitScanned` | `string` | `"Portrait scanned ✓"` | Confirmation after the portrait side is captured. |
| `bothSidesScanned` | `string` | `"Both sides scanned ✓"` | Final confirmation when multi-side scanning completes. |
| `skipPortraitLabel` | `string` | `"Skip portrait scan"` | Label on the skip button shown 5 seconds into the portrait-side phase. |
| `loadImageFailed` | `string` | `"Failed to load image"` | Error shown when an uploaded image cannot be processed. |
| `cameraAccessDenied` | `string` | `"Camera access denied"` | Error shown when the user blocks camera permission. |

#### Example

```ts
const messagesConfig = {
    positionMRZ: "Position the MRZ inside the frame",
    scanSuccess: "Scan complete",
    flipDocument: "Flip the document over",
    flipDocumentCountdown: "Flipping in {seconds}s...",
    cameraAccessDenied: "Please grant camera permission to continue",
};

const mrzScanViewConfig = {
    messagesConfig
};
```

### ThemeConfig

Override the scanner overlay's CSS custom properties for colors, typography, and spacing. Each group is independently optional; set only the tokens you want to override and any unset token falls back to its default.

#### Syntax

```ts
interface ThemeConfig {
  colors?: {
    primary?: string;
    accent?: string;
    backgroundDark?: string;
    backgroundOverlay?: string;
    backgroundSkipLabel?: string;
    text?: string;
    textSecondary?: string;
    divider?: string;
    guideFrame?: string;
    spinnerColor?: string;
    spinnerBackground?: string;
  };

  typography?: {
    fontFamily?: string;
    formatBtnFontSize?: string;
    formatBtnFontSizeSmall?: string;
  };

  spacing?: {
    topBarHeight?: string;
    topBarHeightDesktop?: string;
    guideFrameWidth?: string;
    guideFrameWidthDesktop?: string;
  };
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `colors` | `object` | Palette tokens for the primary UI, accents, backgrounds, text, the guide frame, and the loading spinner. |
| `typography` | `object` | Font family and format-button size tokens. |
| `spacing` | `object` | Top bar height and guide frame width tokens, each with a desktop variant. |

#### Example

```ts
const themeConfig = {
    colors: {
        primary: "#0066cc",
        accent: "#00aaff",
        backgroundDark: "#1a1a1a",
        text: "#ffffff",
        guideFrame: "#ffffff",
    },
    typography: {
        fontFamily: "'Inter', sans-serif",
        formatBtnFontSize: "13px",
    },
    spacing: {
        topBarHeight: "56px",
        guideFrameWidth: "85%",
    },
};

const mrzScanViewConfig = {
    themeConfig
};
```

## Result Interfaces

### MRZResult

The `MRZResult` represents the full result returned by `launch()`. It carries the result `status`, the parsed [`MRZData`](#mrzdata), and three getter methods that expose the captured images.

#### Syntax

```ts
interface MRZResult {
  status?: EnumResultStatus;
  data?: MRZData;

  getDocumentImage(side: EnumDocumentSide): MRZImage | null;
  getOriginalImage(side: EnumDocumentSide): MRZImage | null;
  getPortraitImage(): MRZImage | null;
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `status` | [`EnumResultStatus`]({{ site.api }}enums-mrz-scanner.html#enumresultstatus) | The status of the MRZ result: successful, cancelled (if the user closed the scanner), or failed. |
| `data` | [`MRZData`](#mrzdata) | The parsed MRZ payload. Present on successful scans only. |

#### Methods

| Method | Returns | Description |
| --- | --- | --- |
| `getDocumentImage(side)` | [`MRZImage`](#mrzimage) \| `null` | Returns the deskewed crop of the document for the given side. Requires `returnDocumentImage: true` on `MRZScannerConfig`. Returns `null` if the side wasn't captured (see [`EnumDocumentSide`]({{ site.api }}enums-mrz-scanner.html#enumdocumentside)). |
| `getOriginalImage(side)` | [`MRZImage`](#mrzimage) \| `null` | Returns the full unmodified frame for the given side. Requires `returnOriginalImage: true` on `MRZScannerConfig`. Returns `null` if the side wasn't captured. |
| `getPortraitImage()` | [`MRZImage`](#mrzimage) \| `null` | Returns the cropped portrait, regardless of which side it was found on. Requires `returnPortraitImage: true` on `MRZScannerConfig`. |

#### Example

```ts
(async () => {
    // Launch the scanner and wait for the result
    const result = await mrzScanner.launch();
    console.log(result.status); // prints the result status code to the console
    console.log(result.data); // prints the entire MRZData object to the console

    // Render the cropped document and portrait images
    const documentImage = result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ);
    const portraitImage = result.getPortraitImage();
    if (documentImage) document.body.appendChild(documentImage.toCanvas());
    if (portraitImage) document.body.appendChild(portraitImage.toCanvas());
})();
```

### MRZImage

`MRZImage` extends `DSImageData` with two rendering helpers attached at runtime. It is the value type returned by `getDocumentImage(side)`, `getOriginalImage(side)`, and `getPortraitImage()` on [`MRZResult`](#mrzresult).

#### Syntax

```ts
interface MRZImage extends DSImageData {
  toCanvas(): HTMLCanvasElement;
  toBlob(): Promise<Blob>;
}
```

#### Methods

| Method | Returns | Description |
| --- | --- | --- |
| `toCanvas()` | `HTMLCanvasElement` | Renders the image data into a fresh `HTMLCanvasElement` ready to append to the DOM. |
| `toBlob()` | `Promise<Blob>` | Encodes the image data as a PNG `Blob`. Useful when uploading the image to a server or storing it. |

Beyond these two helpers, `MRZImage` inherits every field of [`DSImageData`]({{ site.dcvb_js_api }}core/basic-structures/ds-image-data.html) (raw bytes, dimensions, pixel format, etc.).

#### Example

```ts
const documentImage = result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ);
if (documentImage) {
    // Render to the DOM
    document.body.appendChild(documentImage.toCanvas());

    // Or upload to a server
    const blob = await documentImage.toBlob();
    await fetch("/upload", { method: "POST", body: blob });
}
```

### MRZData

`MRZData` represents the parsed fields of an MRZ result — names, document number, dates, nationality, and more.

#### Syntax

```ts
interface MRZData {
  [EnumMRZData.InvalidFields]?: EnumMRZData[]; // invalidFields
  [EnumMRZData.DocumentType]: EnumCodeType; // documentType
  [EnumMRZData.DocumentNumber]: string; // documentNumber
  [EnumMRZData.MRZText]: string; // mrzText
  [EnumMRZData.FirstName]: string; // firstName
  [EnumMRZData.LastName]: string; // lastName
  [EnumMRZData.Age]: number; // age
  [EnumMRZData.Sex]: string; // sex
  [EnumMRZData.IssuingState]: string; // issuingState
  [EnumMRZData.IssuingStateRaw]: string; // issuingStateRaw
  [EnumMRZData.Nationality]: string; // nationality
  [EnumMRZData.NationalityRaw]: string; // nationalityRaw
  [EnumMRZData.DateOfBirth]: MRZDate; // dateOfBirth
  [EnumMRZData.DateOfExpiry]: MRZDate; // dateOfExpiry
  [EnumMRZData.OptionalData1]?: string; // optionalData1
  [EnumMRZData.OptionalData2]?: string; // optionalData2
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `invalidFields` | [`EnumMRZData`]({{ site.api }}enums-mrz-scanner.html#enummrzdata) array | List of field keys that failed validation (e.g. failed check-digit verification). |
| `documentType` | `EnumCodeType` | The MRTD code type, as defined by the Dynamsoft Capture Vision bundle (e.g. `CT_MRTD_TD3_PASSPORT`). |
| `documentNumber` | `string` | The MRZ document number. |
| `mrzText` | `string` | The raw unparsed text of the MRZ (line breaks preserved). |
| `firstName` | `string` | The given name(s) of the document holder. |
| `lastName` | `string` | The surname of the document holder. |
| `age` | `number` | The age of the document holder, calculated from `dateOfBirth`. |
| `sex` | `string` | The sex marker of the document holder. |
| `issuingState` | `string` | The issuing state (resolved to the full country/region name). |
| `issuingStateRaw` | `string` | The raw issuing-state code from the MRZ string. |
| `nationality` | `string` | The nationality (resolved to the full country/region name). |
| `nationalityRaw` | `string` | The raw nationality code from the MRZ string. |
| `dateOfBirth` | [`MRZDate`](#mrzdate) | The date of birth of the document holder. |
| `dateOfExpiry` | [`MRZDate`](#mrzdate) | The date of expiry of the document. |
| `optionalData1` | `string` | Optional MRZ-defined data field (present on some formats). |
| `optionalData2` | `string` | Second optional MRZ-defined data field (present on some formats). |

#### Example

```ts
(async () => {
    // Launch the scanner and wait for the result
    const result = await mrzScanner.launch();
    console.log(result.data.firstName); // prints the first name
    console.log(result.data.lastName); // prints the last name
    console.log(result.data.sex); // prints the sex
    console.log(result.data.nationality); // prints the nationality
    console.log(result.data.documentNumber); // prints the document number
})();
```

### MRZDate

`MRZDate` represents the date fields of an `MRZData` object.

#### Syntax

```ts
interface MRZDate {
  year: number;
  month: number;
  day: number;
}
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `year` | `number` | The year of the date (4-digit, e.g. `1992`). |
| `month` | `number` | The month of the date (1–12). |
| `day` | `number` | The day of the date (1–31). |

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

`ResultStatus` is a standalone helper type that pairs an [`EnumResultStatus`]({{ site.api }}enums-mrz-scanner.html#enumresultstatus) code with an optional message. It is exported from the public API for convenience, but the `MRZResult.status` field itself uses the [`EnumResultStatus`]({{ site.api }}enums-mrz-scanner.html#enumresultstatus) value directly.

#### Syntax

```ts
type ResultStatus = {
  code: EnumResultStatus;
  message?: string;
};
```

#### Properties

| Property | Type | Description |
| --- | --- | --- |
| `code` | [`EnumResultStatus`]({{ site.api }}enums-mrz-scanner.html#enumresultstatus) | The status code (`RS_SUCCESS`, `RS_CANCELLED`, or `RS_FAILED`). |
| `message` | `string` | Optional human-readable message accompanying the status. |

## Helpers

The SDK exports a small number of helpers that are useful when building a custom result view.

### displayMRZDate

Formats an [`MRZDate`](#mrzdate) as a `YYYY-MM-DD` string.

#### Syntax

```ts
function displayMRZDate(date: MRZDate): string
```

#### Example

```ts
const dob = Dynamsoft.displayMRZDate(result.data.dateOfBirth);
console.log(dob); // e.g. "1992-04-15"
```

### MRZDataLabel

A constant map that returns a human-readable label for each [`EnumMRZData`]({{ site.api }}enums-mrz-scanner.html#enummrzdata) key. Useful when iterating over `result.data` generically to render a label/value list.

#### Syntax

```ts
const MRZDataLabel: Record<EnumMRZData, string>
```

#### Example

```ts
Object.entries(result.data).forEach(([key, value]) => {
    const label = Dynamsoft.MRZDataLabel[key];
    console.log(`${label}: ${value}`);
});
```

