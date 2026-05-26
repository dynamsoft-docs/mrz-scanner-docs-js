---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Upgrading the MRZ Scanner JavaScript Edition from v3.x to v4.0
keywords: Documentation, MRZ Scanner, Dynamsoft MRZ Scanner JavaScript Edition, Upgrade, Migration, v3.x, v4.0, Breaking Changes
description: A step-by-step migration guide for upgrading a production MRZ Scanner JavaScript Edition integration from v3.x to v4.0.
permalink: /guides/mrz-scanner-upgrade-guide.html
---

<style>
/* Scoped overrides for the checklist <ul>/<ol> below. The template paints */
/* <ul> bullets as ::before pseudo-elements (basis.css), which list-style: none */
/* cannot suppress. Hide the painted marker and disable the real <ol> marker. */
.markdown-body ul.checkbox-list,
.markdown-body ol.checkbox-list {
  list-style: none;
}
.markdown-body ul.checkbox-list li::before {
  content: none;
}
</style>

# Upgrading from v3.x to v4.0

This guide is written for teams already shipping the MRZ Scanner JavaScript Edition on any **v3.x** release and planning to move to **v4.0**. v4.0 is a major release: most config field renames are mechanical, but two architectural shifts (the removal of the built-in result view and a new image-extraction API) require code-level changes that the TypeScript compiler will surface and that your integration tests must cover.

> [!IMPORTANT]
> Do not perform this upgrade as a drop-in dependency bump. Read at least the [TL;DR](#tldr) and the [Migration Checklist](#migration-checklist) end-to-end before changing any code, and exercise the full scan-to-result flow in a staging environment before promoting to production.

If you are starting fresh on v4, ignore this guide and follow the [User Guide]({{ site.guides }}mrz-scanner.html) instead. The v3.x documentation remains available in this site as the `-v3.1` companion files (v3.1 was the last v3.x release, so its docs cover the entire v3.x line). See the [v3.x user guide]({{ site.guides }}mrz-scanner-v3.1.html) and the [v3.x API reference]({{ site.api }}mrz-scanner-v3.1.html).

## TL;DR

The table below ranks every breaking change by impact. Severity ▲ marks architectural changes that require code restructuring; ● marks renames the compiler will catch; ○ marks behavior changes that may silently affect runtime output.

| Severity | Change | Effect on a v3.x integration |
|----------|--------|------------------------------|
| ▲ | [Result view removed](#the-built-in-result-view-is-gone) | `MRZResultView`, `MRZResultViewConfig`, `onDone`, `onCancel`, `showResultView` and the entire result-rendering UI are gone. Your app now renders results. |
| ▲ | [Images via getter methods](#images-are-now-retrieved-via-getter-methods) | `result.originalImageResult` is gone. Use `result.getOriginalImage(side)`, `getDocumentImage(side)`, `getPortraitImage()`. |
| ▲ | [`engineResourcePaths` restructured](#package-dependencies-and-engineresourcepaths) | `rootDirectory` and per-module paths (`std`, `dip`, `core`, …) are replaced by `dcvBundle` + `dcvData`. |
| ○ | [`MRZData.documentType` silently changed](#mrzdatadocumenttype-silently-changed-shape) | Was a humanized string (`"Passport (TD3)"`); now a DCV `EnumCodeType` (`"CT_MRTD_TD3_PASSPORT"`). Any string comparison breaks at runtime, not compile time. |
| ○ | [`EnumMRZDocumentType` string values changed](#enummrzdocumenttype-string-values-changed) | `"passport"` → `"td3_passport"`, `"td1"` → `"td1_id"`, `"td2"` → `"td2_id"`. Enum-reference users are fine; raw-string users break silently. |
| ▲ | [`MRZResult.status` shape changed](#mrzresult-shape-and-status) | Was `{ code, message }` object; now `EnumResultStatus` value directly. `result.status.code` throws. |
| ● | [Scanner view property renames](#mrzscannerviewconfig-property-renames) | `showUploadImage` → `showLoadImageButton`, `uploadAcceptedTypes` → `loadImageAcceptedTypes`, `uploadFileConverter` → `loadImageFileConverter`, `showScanGuide` → `enableScanRegion`. |
| ● | [`EnumMRZScanMode.Passport` → `TD3`](#enummrzscanmode-passport--td3) | Only relevant if you used the scan-mode enum (custom templates). |
| ● | [`cameraEnhancerUIPath` removed](#mrzscannerviewconfig-property-renames) | The v3.x deprecated alias is gone. Use `uiPath`. |
| ○ | [`Dynamsoft` namespace flattened](#dynamsoft-namespace-flattened) | DCV exports are reachable at `Dynamsoft.*`, not `Dynamsoft.Dynamsoft.*`. Anyone deep-importing the SDK namespace breaks. |
| ○ | [Multi-side scanning on by default](#multi-side-scanning-on-by-default) | `returnPortraitImage` defaults to `true`. TD1/TD2 ID scans now prompt for a document flip unless you opt out. |

If your v3.x integration uses only `new MRZScanner({ license })`, awaits `launch()`, and reads `result.data.firstName`-style fields, you may need only the changes in [Package, dependencies, and `engineResourcePaths`](#package-dependencies-and-engineresourcepaths) and [`MRZResult` shape and status](#mrzresult-shape-and-status), plus the section appropriate to your image needs ([Images are now retrieved via getter methods](#images-are-now-retrieved-via-getter-methods)). Otherwise, work through every section below in order.

## Before you upgrade: audit your v3.x integration

Run this list against your codebase before changing anything. Each item maps to a section below, so knowing which apply lets you scope the work.

<ul class="checkbox-list">
  <li><input type="checkbox"> Do you read <code>result.originalImageResult</code>? → <a href="#images-are-now-retrieved-via-getter-methods">Images are now retrieved via getter methods</a></li>
  <li><input type="checkbox"> Do you read <code>result.status.code</code> or <code>result.status.message</code>? → <a href="#mrzresult-shape-and-status"><code>MRZResult</code> shape and status</a></li>
  <li><input type="checkbox"> Do you compare <code>result.data.documentType</code> against any string? → <a href="#mrzdatadocumenttype-silently-changed-shape"><code>MRZData.documentType</code> silently changed shape</a></li>
  <li><input type="checkbox"> Do you pass raw string format names (<code>"passport"</code>, <code>"td1"</code>) to <code>mrzFormatType</code>? → <a href="#enummrzdocumenttype-string-values-changed"><code>EnumMRZDocumentType</code> string values changed</a></li>
  <li><input type="checkbox"> Do you pass any <code>MRZResultViewConfig</code> (e.g. <code>onDone</code>, <code>showOriginalImage</code>, <code>allowResultEditing</code>)? → <a href="#the-built-in-result-view-is-gone">The built-in result view is gone</a></li>
  <li><input type="checkbox"> Do you set <code>showResultView: false</code>? → <a href="#the-built-in-result-view-is-gone">The built-in result view is gone</a></li>
  <li><input type="checkbox"> Do you set <code>cameraEnhancerUIPath</code> anywhere? → <a href="#mrzscannerviewconfig-property-renames"><code>MRZScannerViewConfig</code> property renames</a></li>
  <li><input type="checkbox"> Do you set <code>showUploadImage</code>, <code>uploadAcceptedTypes</code>, <code>uploadFileConverter</code>, or <code>showScanGuide</code>? → <a href="#mrzscannerviewconfig-property-renames"><code>MRZScannerViewConfig</code> property renames</a></li>
  <li><input type="checkbox"> Are you self-hosting DCV engine assets with per-module paths (<code>std</code>, <code>dip</code>, <code>core</code>, …)? → <a href="#package-dependencies-and-engineresourcepaths">Package, dependencies, and <code>engineResourcePaths</code></a></li>
  <li><input type="checkbox"> Do you reach into <code>Dynamsoft.Dynamsoft.*</code> to call DCV directly? → <a href="#dynamsoft-namespace-flattened"><code>Dynamsoft</code> namespace flattened</a></li>
  <li><input type="checkbox"> Do you use <code>utilizedTemplateNames</code> with custom templates? → <a href="#custom-templates-and-utilizedtemplatenames">Custom templates and <code>utilizedTemplateNames</code></a></li>
  <li><input type="checkbox"> Are you scanning TD1 or TD2 ID cards and relying on a single-side capture? → <a href="#multi-side-scanning-on-by-default">Multi-side scanning, on by default</a></li>
</ul>

## The built-in result view is gone

This is the largest behavioral change in v4. v3.x shipped a complete two-stage UI: the camera scanner, then a built-in result view (`MRZResultView`) that showed the parsed fields, the cropped image, edit controls, and Done/Re-scan/Cancel buttons. **v4 removes every part of that.**

### What was removed

- The **`MRZResultView`** class itself (no longer exported).
- The **`MRZResultViewConfig`** interface and the `resultViewConfig` field on `MRZScannerConfig`.
- The **`MRZResultViewToolbarButtonsConfig`** type (which had `cancel` / `rescan` / `done` keys).
- The **`showResultView`** boolean toggle on `MRZScannerConfig`.
- The **`onDone`** and **`onCancel`** callbacks.
- Every result-view-only flag: **`showOriginalImage`**, **`showMRZText`**, **`allowResultEditing`**, **`emptyResultMessage`**.

### What replaces it

`launch()` returns a `Promise<MRZResult>` directly. Your application is responsible for rendering the result however your design system requires. The promise resolves once the scanner closes. There is no intermediate review stage owned by the SDK.

### Migration recipe

**v3.x (relying on the built-in result view and `onDone`):**

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
  resultViewConfig: {
    showOriginalImage: true,
    allowResultEditing: true,
    onDone: async (result) => {
      console.log(result.status.code);
      console.log(result.data?.firstName);
      await submitToServer(result.data);
    },
    onCancel: async () => {
      navigateHome();
    },
  },
});

await mrzScanner.launch();
```

**v4 (handling the promise yourself):**

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
  // returnDocumentImage and returnPortraitImage default to true
  // see the image-getter section below
});

const result = await mrzScanner.launch();

if (!result?.data) {
  // User cancelled, or no MRZ was found in a static image.
  navigateHome();
  return;
}

console.log(result.status); // EnumResultStatus value directly (not result.status.code)
console.log(result.data.firstName);
await submitToServer(result.data);
// Render the parsed fields, the cropped image, and any edit affordances yourself.
```

> [!NOTE]
> If your v3.x app relied on `allowResultEditing` to let users correct parsed fields against the physical document, that flow is now entirely yours to build. The parsed `MRZData` exposes the same fields it always did (plus `optionalData1` / `optionalData2`); you decide how to present and persist edits.

### Cancellation detection

In v3.x, `result.status.code === Dynamsoft.EnumResultStatus.RS_CANCELLED` distinguished cancellation from success. In v4, the canonical pattern is to check that `result?.data` exists:

```ts
const result = await mrzScanner.launch();
if (!result?.data) {
  // Cancelled, or static-image input contained no detectable MRZ.
  return;
}
```

`result.status` is still present and is now an `EnumResultStatus` value (not a wrapper object). Use it if you need to distinguish cancellation from no-MRZ-found. See [`MRZResult` shape and status](#mrzresult-shape-and-status) for the full shape.

## Images are now retrieved via getter methods

v3.x attached a single image to the result as a property:

```ts
// v3.x
const dsImageData = result.originalImageResult; // DSImageData | undefined
```

v4 replaces this single property with three getter methods that return a `MRZImage` (a `DSImageData` with extra `toCanvas()` / `toBlob()` helpers) per requested side:

```ts
// v4
result.getOriginalImage(side: EnumDocumentSide): MRZImage | null
result.getDocumentImage(side: EnumDocumentSide): MRZImage | null
result.getPortraitImage(): MRZImage | null
```

Three things are different:

1. **Per-image opt-in.** v4 introduces three flags on `MRZScannerConfig` that control which images are produced and attached to the result. Set them on the constructor config:

    | Flag | Default | Retrieve with |
    |------|---------|----------------|
    | `returnOriginalImage` | `false` | `result.getOriginalImage(side)` |
    | `returnDocumentImage` | `true`  | `result.getDocumentImage(side)` |
    | `returnPortraitImage` | `true`  | `result.getPortraitImage()` |

    The v3.x equivalent of `result.originalImageResult` is `result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ)`, but only if you opt in by setting `returnOriginalImage: true`.

2. **`EnumDocumentSide` is new.** `MRZ` selects the side that contains the MRZ text; `Opposite` selects the flipped side (only populated when multi-side scanning ran; see [multi-side scanning](#multi-side-scanning-on-by-default)).

3. **`MRZImage` ships with helpers.** Each non-null image has `.toCanvas()` and `.toBlob()`. In v3.x you had to convert `DSImageData` yourself.

### Migration recipe

**v3.x:**

```ts
const result = await mrzScanner.launch();
const canvas = result.originalImageResult?.toCanvas();
if (canvas) imageContainer.appendChild(canvas);
```

**v4 (single-side, the closest behavioral match to v3.x):**

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
  returnOriginalImage: true,         // opt in (defaults to false)
  returnDocumentImage: true,         // deskewed crop, on by default
  returnPortraitImage: false,        // disable multi-side scanning if you don't need the portrait
});

const result = await mrzScanner.launch();

const original = result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ);
const docCrop = result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ);

if (original) imageContainer.appendChild(original.toCanvas());
if (docCrop)  cropContainer.appendChild(docCrop.toCanvas());
```

**v4 (multi-side, when scanning TD1/TD2 IDs and you want the portrait):**

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
  returnDocumentImage: true,         // default — include the deskewed crop on each side
  returnPortraitImage: true,         // default — activates the flip prompt for TD1/TD2
});

const result = await mrzScanner.launch();

const portrait             = result.getPortraitImage();
const mrzSideProcessed     = result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ);
const oppositeSideProcessed = result.getDocumentImage(Dynamsoft.EnumDocumentSide.Opposite);
```

> [!WARNING]
> Any getter call returns `null` when the corresponding image was not requested (via the `return*Image` flags) or could not be captured (for example, `getDocumentImage(Opposite)` on a passport, where the portrait sits on the MRZ side). Guard every call before touching the result.

### Removed undocumented fields

v3.x had two undocumented fields on `MRZResult` used by Dynamsoft Mobile Web Capture for interop: `imageData?: boolean` and `_imageData?: DSImageData`. Both are removed in v4. They were never part of the public surface, but if your code reaches into them, replace those reads with the appropriate getter call.

## Package, dependencies, and `engineResourcePaths`

The npm package name is unchanged (`dynamsoft-mrz-scanner`); only the version bumps to `4.0.0`. The peer-dependency packages and their layout under `node_modules/` did change.

### Bump the dependency

```sh
npm i dynamsoft-mrz-scanner@4.0.0 -E
# or
yarn add dynamsoft-mrz-scanner@4.0.0 -E
```

This installs two peer dependencies automatically: **`dynamsoft-capture-vision-bundle`** (the DCV engine: JavaScript plus WebAssembly) and **`dynamsoft-capture-vision-data`** (the DCV model, template, and parser data files). Together they replace the multiple per-module DCV packages that v3.x referenced individually.

The expected `node_modules/` layout after install:

```
node_modules/
├── dynamsoft-mrz-scanner/              # the library bundle and UI/template assets
├── dynamsoft-capture-vision-bundle/    # DCV engine (JS + WASM)
└── dynamsoft-capture-vision-data/      # DCV model, template, and parser data
```

### CDN URL

If you load the SDK from a CDN, update the version segment of the URL:

```html
<!-- v3.x -->
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@3.1.0/dist/mrz-scanner.bundle.js"></script>

<!-- v4 -->
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@4.0.0/dist/mrz-scanner.bundle.js"></script>
```

### Recommended: stage the folders and drop `engineResourcePaths` entirely

v4's default resource resolution is built around the assumption that the three Dynamsoft folders are served at the same origin as your page, at `/`. Stage them into your project's `public/` directory and **you can delete `engineResourcePaths` from your config entirely**. The SDK's defaults find every resource.

```bash
mkdir public
cp -R node_modules/dynamsoft-mrz-scanner \
      node_modules/dynamsoft-capture-vision-bundle \
      node_modules/dynamsoft-capture-vision-data public/
```

```ts
// v4 — with the folders staged in public/, this is sufficient:
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
});
```

> [!TIP]
> Re-run the copy after every `npm install` that bumps the SDK or DCV (or script it into your build).

### Or: keep `engineResourcePaths` and restructure the shape

If you need pinned hosting (a CDN, a `/static/` prefix, an internal mirror), keep `engineResourcePaths` and migrate its shape. v3.x forwarded the option verbatim to the DCV `CoreModule`, which historically accepted **two different shapes**. You need to know which one you used.

**Shape A: single root directory.** This is what the v3.x default and most quick-starts used:

```ts
// v3.x
engineResourcePaths: {
  rootDirectory: "https://cdn.jsdelivr.net/npm/",
}
```

**Shape B: per-module paths.** Some self-hosting integrations pinned every DCV module individually:

```ts
// v3.x
engineResourcePaths: {
  std:     "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-std/dist/",
  dip:     "https://cdn.jsdelivr.net/npm/dynamsoft-image-processing/dist/",
  core:    "https://cdn.jsdelivr.net/npm/dynamsoft-core/dist/",
  license: "https://cdn.jsdelivr.net/npm/dynamsoft-license/dist/",
  cvr:     "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-router/dist/",
  dlr:     "https://cdn.jsdelivr.net/npm/dynamsoft-label-recognizer/dist/",
  dcp:     "https://cdn.jsdelivr.net/npm/dynamsoft-code-parser/dist/",
}
```

**v4 collapses both shapes into two paths** pointing at the bundled DCV peers, wherever you serve them from:

```ts
// v4
engineResourcePaths: {
  dcvBundle: "https://cdn.example.com/static/dynamsoft-capture-vision-bundle/dist/",
  dcvData:   "https://cdn.example.com/static/dynamsoft-capture-vision-data/",
}
```

The requirement is just that the contents of those two npm packages are reachable at the URLs you provide. If you previously had Webpack / Vite / Rollup config that copied per-module DCV directories into `dist/`, simplify it to copy only `dynamsoft-capture-vision-bundle` and `dynamsoft-capture-vision-data`.

## `MRZResult` shape and status

The result object's shape changed in two ways. The status field is the more dangerous one because the v3.x access pattern (`result.status.code`) throws at runtime in v4 instead of failing at compile time.

### `MRZResult` field-by-field

| v3.x | v4 | Notes |
|------|----|-------|
| `status: ResultStatus` (required `{ code, message }` object) | `status?: EnumResultStatus` (optional enum value) | Shape changed and field is now optional. See below. |
| `originalImageResult?: DSImageData` | *(removed)* | Use `getOriginalImage(side)`. See [Images are now retrieved via getter methods](#images-are-now-retrieved-via-getter-methods). |
| `data?: MRZData` | `data?: MRZData` | Unchanged at this level; the `MRZData` interface itself gained two fields. See [`MRZData.documentType` silently changed shape](#mrzdatadocumenttype-silently-changed-shape). |
| *(none)* | `getDocumentImage(side): MRZImage \| null` | New. |
| *(none)* | `getOriginalImage(side): MRZImage \| null` | New. |
| *(none)* | `getPortraitImage(): MRZImage \| null` | New. |
| `imageData?: boolean` (undocumented) | *(removed)* | Was MWC interop. |
| `_imageData?: DSImageData` (undocumented) | *(removed)* | Was MWC interop. |

### The `status` change

v3.x:

```ts
// v3.x
interface MRZResult {
  status: ResultStatus;   // { code: EnumResultStatus; message?: string }
  // ...
}

// Usage
if (result.status.code === Dynamsoft.EnumResultStatus.RS_CANCELLED) { /* ... */ }
console.log(result.status.message);
```

v4:

```ts
// v4
interface MRZResult {
  status?: EnumResultStatus;   // the enum value directly, not an object
  // ...
}

// Usage
if (result.status === Dynamsoft.EnumResultStatus.RS_CANCELLED) { /* ... */ }
// There is no result.status.message in v4. If you need a human-readable status,
// produce it yourself from the enum value.
```

The `EnumResultStatus` enum itself is unchanged (`RS_SUCCESS = 0`, `RS_CANCELLED = 1`, `RS_FAILED = 2`).

> [!WARNING]
> If your v3.x code did `result.status.code` or `result.status.message`, those reads throw `TypeError` in v4 because `status` is now a number, not an object. Search-replace `result.status.code` → `result.status`, and remove any reference to `result.status.message`.

### A `ResultStatus` type still exists, but it's not what `MRZResult.status` uses

v4 still exports a standalone `ResultStatus = { code: EnumResultStatus; message?: string }` type as a helper. **It is not the type of `MRZResult.status`**. That field is now `EnumResultStatus` directly. If you imported `ResultStatus` only to type the result's status field, drop the import.

## `MRZData.documentType` silently changed shape

This change does not produce a TypeScript error and does not throw at runtime. It just returns different values from what your v3.x code expected. **Audit any code that compares `result.data.documentType` against a string literal.**

### v3.x behavior

`MRZData.documentType: string` returned a humanized display label produced by the internal parser. The values your application saw were strings like:

- `"Passport (TD3)"`
- `"ID (TD1)"`
- `"ID (TD2)"`
- `"ID (VISA)"`
- `"French ID (TD2)"`
- `"Visa (TD3)"`

These were intended for direct display in the built-in result view. Some integrations branched on them:

```ts
// v3.x — works, returns a display label
if (result.data.documentType.startsWith("Passport")) {
  // ...
}
```

### v4 behavior

`MRZData.documentType: EnumCodeType` returns the DCV code-type identifier, which is the underlying enum value, not a display label:

- `"CT_MRTD_TD3_PASSPORT"`
- `"CT_MRTD_TD1_ID"`
- `"CT_MRTD_TD2_ID"`
- `"CT_MRTD_TD3_VISA"`
- `"CT_MRTD_TD2_VISA"`

```ts
// v4 — same field, different shape
if (result.data.documentType === "CT_MRTD_TD3_PASSPORT") {
  // ...
}
```

### Migration recipe

If you previously displayed `result.data.documentType` directly to users, replace that with your own mapping from `EnumCodeType` (or from `EnumMRZDocumentType`; see [`EnumMRZDocumentType` string values changed](#enummrzdocumenttype-string-values-changed)) to a localized display label. The new `MRZDataLabel` helper handles field-key labels but **not** document-type labels, so those are yours to provide.

If you branched on the value, swap the string comparisons:

| v3.x value (humanized) | v4 value (`EnumCodeType`) |
|------------------------|---------------------------|
| `"Passport (TD3)"` | `"CT_MRTD_TD3_PASSPORT"` |
| `"ID (TD1)"` | `"CT_MRTD_TD1_ID"` |
| `"ID (TD2)"`, `"French ID (TD2)"` | `"CT_MRTD_TD2_ID"` |
| `"Visa (TD3)"`, `"ID (VISA)"` (TD3-sized) | `"CT_MRTD_TD3_VISA"` |
| `"Visa (TD2)"`, `"ID (VISA)"` (TD2-sized) | `"CT_MRTD_TD2_VISA"` |

### Two new `MRZData` fields

v4 surfaces two ICAO optional-data fields that v3.x did not expose:

```ts
[EnumMRZData.OptionalData1]?: string; // optionalData1
[EnumMRZData.OptionalData2]?: string; // optionalData2
```

These are additive, so existing code is unaffected. They appear on `EnumMRZData` as `OptionalData1` and `OptionalData2`.

`IssuingStateRaw` and `NationalityRaw` were already present on `MRZData` in v3.x (added in v2.1); v4 promotes them to the documented surface but the values themselves are unchanged.

## `EnumMRZDocumentType` string values changed

The enum still exists and still has the same TypeScript-side names, but the underlying string values changed to disambiguate the new visa formats.

```ts
// v3.x
enum EnumMRZDocumentType {
  Passport = "passport",
  TD1 = "td1",
  TD2 = "td2",
}

// v4
enum EnumMRZDocumentType {
  Passport = "td3_passport",
  TD1 = "td1_id",
  TD2 = "td2_id",
  MRVA = "mrva_visa",   // NEW — TD3-sized machine-readable visa
  MRVB = "mrvb_visa",   // NEW — TD2-sized machine-readable visa
}
```

### When this affects you

- **You used enum references** (`Dynamsoft.EnumMRZDocumentType.Passport`): **no change required.** The enum members still exist and still mean the same thing.
- **You passed raw strings to `mrzFormatType`** (`mrzFormatType: ["passport", "td1"]`): **breaks silently.** v4 does not recognize the old values. Update to `["td3_passport", "td1_id"]`.
- **You compared against strings** elsewhere: same problem. Update the literals.

### Quick string conversion

| v3.x string | v4 string |
|-------------|-----------|
| `"passport"` | `"td3_passport"` |
| `"td1"` | `"td1_id"` |
| `"td2"` | `"td2_id"` |
| *(no equivalent)* | `"mrva_visa"` |
| *(no equivalent)* | `"mrvb_visa"` |

### v3.x conflated visas with passports/TD2s

In v3.x, the scanner internally recognized visa code-types but mapped them back onto `Passport` or `TD2` for the purposes of `EnumMRZDocumentType`. If your v3.x app received `Passport` results that were actually visas, those will now arrive as `MRVA` (TD3-sized) or `MRVB` (TD2-sized) instead. This is a behavioral improvement, but worth flagging if your downstream pipeline assumed every `Passport` was a true passport.

## `EnumMRZScanMode`: `Passport` → `TD3`

`EnumMRZScanMode` is the broader internal mode enum that includes combined modes (`PassportAndTD1`, etc.). Only relevant if you used it to key into `utilizedTemplateNames` for custom templates.

The breaking rename: **`EnumMRZScanMode.Passport` is now `EnumMRZScanMode.TD3`**, matching the rest of the v4 naming. Other members are unchanged.

If you have a `utilizedTemplateNames` object keyed by `EnumMRZScanMode.Passport`, rename the key to `EnumMRZScanMode.TD3`.

## `MRZScannerViewConfig` property renames

Four renames in the scanner view config. Each is mechanical. Your TypeScript compiler will catch them if you're using types, but plain-JS callers should grep their source.

| v3.x | v4 | Notes |
|------|----|-------|
| `showUploadImage` | `showLoadImageButton` | Same default (`true`) and same behavior. |
| `uploadAcceptedTypes` | `loadImageAcceptedTypes` | Same default (`"image/*"`). |
| `uploadFileConverter` | `loadImageFileConverter` | Same signature: `(file: File) => Promise<Blob>`. |
| `showScanGuide` | `enableScanRegion` | Same default (`true`). The semantic shift is that the option now describes the *region* the scanner reads from, not just the visual overlay. The two were always coupled, but v4 makes the naming reflect that. |
| `cameraEnhancerUIPath` | `uiPath` | `uiPath` was already the canonical name in v3.x; `cameraEnhancerUIPath` was a deprecated alias. v4 removes the alias. |

### `MRZScannerConfig` removed/changed fields

At the top level (the constructor's config object):

| v3.x | v4 | Notes |
|------|----|-------|
| `resultViewConfig` | *(removed)* | See [The built-in result view is gone](#the-built-in-result-view-is-gone). |
| `showResultView` | *(removed)* | See [The built-in result view is gone](#the-built-in-result-view-is-gone). |
| *(none)* | `returnOriginalImage`, `returnDocumentImage`, `returnPortraitImage` | New image-output flags. See [Images are now retrieved via getter methods](#images-are-now-retrieved-via-getter-methods). |

`templateFilePath` and `utilizedTemplateNames` are still supported. See [Custom templates and `utilizedTemplateNames`](#custom-templates-and-utilizedtemplatenames).

### `MRZScannerViewConfig` additive fields

v4 also adds several new optional fields. None of these break v3.x code, but they offer customization options you may have implemented manually:

- **`flipDocumentTimeout`** (default `3000` ms): controls the pause between MRZ and portrait capture during multi-side scanning.
- **`toolbarButtonsConfig`**: per-button override for the seven scanner toolbar buttons (`close`, `loadImage`, `cameraSwitch`, `flash`, `flashOff`, `sound`, `soundOff`). Not to be confused with the v3.x *result-view* `toolbarButtonsConfig` (which had `cancel`/`rescan`/`done` keys and is gone).
- **`formatSelectorConfig`**: labels for the four format selector buttons.
- **`messagesConfig`**: every on-screen text string, including new multi-side messages (`flipDocument`, `flipDocumentCountdown` with a `{seconds}` placeholder, etc.).
- **`themeConfig`**: CSS tokens for colors, typography, and spacing on the scanner overlay.

See the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html) for the full options.

## `launch()` signature

The `launch()` signature is effectively unchanged. v3.x already accepted the wide input union; v4 keeps it. Listed here only because the v3.x API docs understated the parameter type.

```ts
// v3.x actual signature (source)
async launch(
  imageOrFile?: Blob | string | DSImageData | HTMLImageElement | HTMLVideoElement | HTMLCanvasElement
): Promise<MRZResult>

// v4 signature
launch(): Promise<MRZResult>
launch(imageSource: Blob | string | DSImageData | HTMLImageElement | HTMLVideoElement | HTMLCanvasElement): Promise<MRZResult>
```

The only thing to note: v4 documents that `launch()` calls `dispose()` automatically in its `finally` block on every resolution path. `dispose()` is idempotent, so only call it manually if you want to tear down the scanner without ever launching it. v3.x documented `dispose()` more loosely.

## `Dynamsoft` namespace flattened

v3.x published its bundle with a double-nested `Dynamsoft.Dynamsoft.*` namespace path for reaching into the underlying DCV SDK. v4 flattens this: DCV exports are now reachable directly under `Dynamsoft.*`.

```ts
// v3.x — deep DCV access
const router = await Dynamsoft.Dynamsoft.CVR.CaptureVisionRouter.createInstance();

// v4 — flat
const router = await Dynamsoft.CVR.CaptureVisionRouter.createInstance();
```

The MRZ Scanner's own public exports (`MRZScanner`, `MRZScannerView`, the enums, `MRZDataLabel`, `displayMRZDate`) sit at `Dynamsoft.*` in both versions. No change for ordinary integrations. The flattening only matters if you reach past the MRZ Scanner surface into DCV directly.

If you only use `import { Dynamsoft } from "dynamsoft-mrz-scanner"` (or the global `Dynamsoft.MRZScanner` on the CDN bundle), nothing changes. If you grep your code for `Dynamsoft.Dynamsoft` and find no matches, you're done with this section.

## Custom templates and `utilizedTemplateNames`

If you do **not** use `templateFilePath` or `utilizedTemplateNames`, skip this section.

For integrations with custom Capture Vision templates:

- `templateFilePath` is unchanged.
- `utilizedTemplateNames` is still supported. The v4 type widens each value: it now accepts either a `string` (the v3.x shape) or a `TemplatePair` (`{ full, mrzOnly }`) per scan mode. The string form continues to work, with existing keys keeping their original semantics. Use the pair form only if you want separate templates for full-frame vs MRZ-only capture.
- The `EnumMRZScanMode` keys are unchanged, **except** `Passport` was renamed to `TD3`. See [`EnumMRZScanMode`: `Passport` → `TD3`](#enummrzscanmode-passport--td3).
- `DEFAULT_TEMPLATE_NAMES` (the default scan-mode → template-name map exported by the package) is still available.

If you maintain a non-trivial custom template setup, contact the [Dynamsoft Technical Support Team](https://www.dynamsoft.com/company/contact/) for help validating your migration; the underlying DCV runtime upgrade in v4 (DCV 3.4.2001) may interact with parameter-level customizations that aren't visible in the MRZ Scanner surface.

## New v4 features worth knowing about

These are additive and do not break v3.x code, but several materially change default runtime behavior. Read at least the [multi-side scanning](#multi-side-scanning-on-by-default) section before deploying.

### Multi-side scanning, on by default

`returnPortraitImage` defaults to `true`. For passports (TD3), the portrait sits on the same side as the MRZ, so the scan completes in one capture as before. For **TD1 and TD2 ID cards**, the portrait is on the opposite side of the MRZ, so the scanner now prompts the user to flip the document after the MRZ side is captured, then captures the portrait side. The total session takes longer and produces two images per side (deskewed crop plus optional original).

If your existing v3.x flow assumes a single capture per scan (for example, your UI shows a "Scanning…" spinner that you dismiss as soon as `launch()` returns), the longer multi-side flow may surprise users on ID-card scans. Three options:

- **Keep the new behavior.** Update your UI to reflect that ID-card scans take longer and produce a portrait image alongside the MRZ-side image.
- **Disable multi-side scanning entirely.** Set `returnPortraitImage: false` on the constructor config. The scanner now stops after the MRZ side, matching v3.x behavior, and `getPortraitImage()` / `getDocumentImage(Opposite)` always return `null`.
- **Tune the flip timeout.** Default is 3000 ms; lower it via `scannerViewConfig.flipDocumentTimeout` if your testing shows users flip faster than expected.

### Theme, messages, format-selector customization

If you previously customized the scanner UI by editing the bundled HTML template, the new `themeConfig` / `messagesConfig` / `formatSelectorConfig` / `toolbarButtonsConfig` interfaces may let you do it from JavaScript instead, without forking assets. See the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).

### `displayMRZDate` and `MRZDataLabel` helpers

Both were present in v3.x but rarely needed (the built-in result view handled date formatting and field labels for you). Now that you own the result view, these helpers become useful:

```ts
const dob = Dynamsoft.displayMRZDate(result.data.dateOfBirth);   // "1985-07-12"
const label = Dynamsoft.MRZDataLabel[Dynamsoft.EnumMRZData.FirstName]; // "First Name"
```

Note that `MRZDataLabel` provides labels for field *keys*, not for `documentType` values. See [`MRZData.documentType` silently changed shape](#mrzdatadocumenttype-silently-changed-shape) for that.

## Static-image scanning migration

If your v3.x app uses the static-image (file / PDF / blob) flow via `launch(file)`, the migration is small. The two property renames in [`MRZScannerViewConfig` property renames](#mrzscannerviewconfig-property-renames) apply:

```ts
// v3.x
scannerViewConfig: {
  uploadAcceptedTypes: "image/*,application/pdf",
  uploadFileConverter: async (file) => { /* ... */ },
}

// v4
scannerViewConfig: {
  loadImageAcceptedTypes: "image/*,application/pdf",
  loadImageFileConverter: async (file) => { /* ... */ },
}
```

The `launch(file)` call itself is unchanged. Image retrieval from a static-image scan follows the same getter pattern as a camera scan: use `result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ)` to get the input image back. Multi-side scanning does not apply to static images (there's only one input).

See the [Reading Static Images guide]({{ site.guides }}mrz-scanner-static-image.html) for the full v4 walkthrough.

## End-to-end before/after

A complete, realistic v3.x wire-up alongside the v4 equivalent. Read the deltas closely. Every change traces back to a section above.

### v3.x

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
  mrzFormatType: ["passport", "td1"],
  engineResourcePaths: {
    rootDirectory: "https://cdn.jsdelivr.net/npm/",
  },
  scannerViewConfig: {
    showScanGuide: true,
    showUploadImage: true,
    uploadAcceptedTypes: "image/*,application/pdf",
    uploadFileConverter: async (file) => convertPdfToBlob(file),
  },
  resultViewConfig: {
    showOriginalImage: true,
    showMRZText: true,
    allowResultEditing: true,
    onDone: async (result) => {
      if (result.status.code !== Dynamsoft.EnumResultStatus.RS_SUCCESS) return;
      console.log(result.data.firstName);
      console.log(result.data.documentType);   // "Passport (TD3)"
      const canvas = result.originalImageResult?.toCanvas();
      if (canvas) imageContainer.appendChild(canvas);
      await submitToServer(result.data);
    },
    onCancel: async () => navigateHome(),
  },
});

await mrzScanner.launch();
```

### v4

```ts
// Stage the three Dynamsoft folders into your project's `public/` directory
// so the SDK's defaults resolve every engine resource. No engineResourcePaths needed.
const mrzScanner = new Dynamsoft.MRZScanner({
  license: "YOUR_LICENSE_KEY_HERE",
  mrzFormatType: ["td3_passport", "td1_id"],            // string values changed in v4
  returnOriginalImage: true,                             // opt in for the original frame
  // returnDocumentImage / returnPortraitImage default to true
  scannerViewConfig: {
    enableScanRegion: true,                              // renamed from showScanGuide
    showLoadImageButton: true,                           // renamed from showUploadImage
    loadImageAcceptedTypes: "image/*,application/pdf",   // renamed from uploadAcceptedTypes
    loadImageFileConverter: async (file) => convertPdfToBlob(file), // renamed from uploadFileConverter
  },
  // resultViewConfig removed in v4
});

const result = await mrzScanner.launch();

if (!result?.data) {                                     // cancellation or no-MRZ check
  navigateHome();
  return;
}

console.log(result.data.firstName);
console.log(result.data.documentType);                   // "CT_MRTD_TD3_PASSPORT" (was a humanized label in v3.x)
const original = result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ); // image getter replaces originalImageResult
if (original) imageContainer.appendChild(original.toCanvas());
await submitToServer(result.data);

// If you previously had allowResultEditing: true, build that UI yourself now.
```

## Migration checklist

Copy this into your tracking system and work through it linearly.

<ol class="checkbox-list">
  <li><input type="checkbox"> 1. Bump <code>dynamsoft-mrz-scanner</code> to <code>4.0.0</code>; remove any v3.x per-module DCV dependencies; verify the two peer packages installed.</li>
  <li><input type="checkbox"> 2. Update CDN URLs to <code>@4.0.0</code> if applicable.</li>
  <li><input type="checkbox"> 3. Stage the three Dynamsoft folders (<code>dynamsoft-mrz-scanner</code>, <code>dynamsoft-capture-vision-bundle</code>, <code>dynamsoft-capture-vision-data</code>) into your project's <code>public/</code> directory so they're served at <code>/</code>. With the folders staged, <strong>delete <code>engineResourcePaths</code> from your config</strong>. Only keep it (in the new <code>{ dcvBundle, dcvData }</code> shape) if you must serve the folders from a different origin or path prefix.</li>
  <li><input type="checkbox"> 4. Remove <code>resultViewConfig</code>, <code>showResultView</code>, <code>onDone</code>, <code>onCancel</code> from your constructor config. Move the work each callback did into code that runs after <code>await mrzScanner.launch()</code>.</li>
  <li><input type="checkbox"> 5. Build a result UI in your application: parsed fields, document image, portrait image (if you keep <code>returnPortraitImage: true</code>), and any re-scan / done / cancel affordances your design system requires.</li>
  <li><input type="checkbox"> 6. Replace <code>result.originalImageResult</code> reads with <code>result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ)</code> (and add <code>returnOriginalImage: true</code> to your config if you need this image).</li>
  <li><input type="checkbox"> 7. If you display the deskewed document crop, use <code>result.getDocumentImage(side)</code>. If you display the portrait, use <code>result.getPortraitImage()</code>.</li>
  <li><input type="checkbox"> 8. Replace <code>result.status.code</code> with <code>result.status</code>. Replace <code>result.status.message</code> with your own enum-to-string mapping.</li>
  <li><input type="checkbox"> 9. Use <code>if (!result?.data)</code> as your cancellation / no-result branch.</li>
  <li><input type="checkbox"> 10. Search the codebase for raw string format names (<code>"passport"</code>, <code>"td1"</code>, <code>"td2"</code>) and update them (<code>"td3_passport"</code>, <code>"td1_id"</code>, <code>"td2_id"</code>). Prefer enum references over strings going forward.</li>
  <li><input type="checkbox"> 11. Search the codebase for comparisons against <code>result.data.documentType</code>. The values are now <code>CT_MRTD_*</code> code types, not display labels. Update comparisons or compute display labels from a new mapping.</li>
  <li><input type="checkbox"> 12. Rename <code>showUploadImage</code> → <code>showLoadImageButton</code>, <code>uploadAcceptedTypes</code> → <code>loadImageAcceptedTypes</code>, <code>uploadFileConverter</code> → <code>loadImageFileConverter</code>, <code>showScanGuide</code> → <code>enableScanRegion</code>, <code>cameraEnhancerUIPath</code> → <code>uiPath</code> in any <code>scannerViewConfig</code> you pass.</li>
  <li><input type="checkbox"> 13. If you key custom templates by <code>EnumMRZScanMode.Passport</code>, rename to <code>EnumMRZScanMode.TD3</code>.</li>
  <li><input type="checkbox"> 14. Search for <code>Dynamsoft.Dynamsoft.</code> and flatten any matches to <code>Dynamsoft.</code>.</li>
  <li><input type="checkbox"> 15. Decide on multi-side scanning. If you scan TD1 / TD2 ID cards and want to preserve v3.x's single-capture behavior, set <code>returnPortraitImage: false</code>. Otherwise, update your scanner-active UI to reflect the longer flip-and-capture flow.</li>
  <li><input type="checkbox"> 16. Run your full integration test suite. TypeScript will catch the renames, but the runtime changes (<code>result.status</code>, <code>documentType</code> values, raw format strings) will only surface in test or in production.</li>
  <li><input type="checkbox"> 17. Smoke-test the camera flow and (if applicable) the static-image / PDF flow against real documents.</li>
</ol>

## Further reading

- **[v4 User Guide]({{ site.guides }}mrz-scanner.html)**: the canonical v4 walkthrough: npm-based install, self-hosting the runtime assets under `public/`, and a minimal result view (canvases + JSON dump) that exercises every getter your app now owns.
- **[v4 Quick Start]({{ site.guides }}mrz-scanner-quick-start.html)**: single-file Hello World loaded from a CDN. Useful as a fast sanity check of the new v4 API against your license key before tackling the full migration.
- **[MRZ Scanner Demo]({{ site.codegallery }}demo/index.html)**: fully styled, mobile-and-desktop-responsive reference implementation. Since v4 removes the built-in result view, this is the closest reference for the polished result-rendering UI your app needs to build.
- **[v4 Customization Guide]({{ site.guides }}mrz-scanner-customization.html)**: `toolbarButtonsConfig`, `messagesConfig`, `themeConfig`, `formatSelectorConfig`, and multi-side scanning details.
- **[v4 Static Image Guide]({{ site.guides }}mrz-scanner-static-image.html)**: the file / PDF / blob flow under v4.
- **[v4 API Reference]({{ site.api }}mrz-scanner.html)**: full type definitions for `MRZScannerConfig`, `MRZScannerViewConfig`, `MRZResult`, and every sub-interface mentioned above.
- **[v4 Enums Reference]({{ site.api }}enums-mrz-scanner.html)**: `EnumMRZDocumentType`, `EnumDocumentSide`, `EnumMRZData`, `EnumResultStatus`.
- **[v3.x User Guide]({{ site.guides }}mrz-scanner-v3.1.html)** and **[v3.x API Reference]({{ site.api }}mrz-scanner-v3.1.html)**: preserved for reference during the migration.
- **[Release Notes]({{ site.releasenotes }}index.html)**: the full v4.0.0 changelog.

If you hit a migration scenario this guide doesn't cover, contact the [Dynamsoft Support Team](https://www.dynamsoft.com/company/contact/), particularly for custom-template setups, framework-specific build configurations, or Mobile Web Capture integration paths.
