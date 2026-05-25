---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: MRZ Scanner JavaScript Edition - FAQ
keywords: Documentation, MRZ Scanner JavaScript Edition, FAQ, Frequently Asked Questions
breadcrumbText: FAQ
description: MRZ Scanner JavaScript Edition Documentation Frequently Asked Questions
permalink: /faq/index.html
---

# Frequently Asked Questions

This page collects answers to common questions about the MRZ Scanner JavaScript Edition. For step-by-step walkthroughs see the [User Guide]({{ site.guides }}mrz-scanner.html); for migrating from v3.x see the [Upgrade Guide]({{ site.guides }}mrz-scanner-upgrade-guide.html).

**Capability and supported formats**

- [Can the MRZ Scanner process static images, or does it only work with live cameras?](#can-the-mrz-scanner-process-static-images-or-does-it-only-work-with-live-cameras)
- [Can the MRZ Scanner read PDFs?](#can-the-mrz-scanner-read-pdfs)
- [Which document types and MRZ formats does the MRZ Scanner recognize?](#which-document-types-and-mrz-formats-does-the-mrz-scanner-recognize)
- [Does the MRZ Scanner work on mobile browsers?](#does-the-mrz-scanner-work-on-mobile-browsers)
- [Does the MRZ Scanner work with React, Vue, or Angular?](#does-the-mrz-scanner-work-with-react-vue-or-angular)

**Privacy and data handling**

- [Does the MRZ Scanner perform data validation?](#does-the-mrz-scanner-perform-data-validation)
- [Does the MRZ Scanner work offline, and does it send any data to Dynamsoft?](#does-the-mrz-scanner-work-offline-and-does-it-send-any-data-to-dynamsoft)

**Defaults and gotchas**

- [Why does scanning a TD1 or TD2 ID card prompt the user to flip the document?](#why-does-scanning-a-td1-or-td2-id-card-prompt-the-user-to-flip-the-document)
- [Why does the document type now return a code like CT_MRTD_TD3_PASSPORT instead of a readable label?](#why-does-the-document-type-now-return-a-code-like-ct_mrtd_td3_passport-instead-of-a-readable-label)
- [Why do I get a resource-initialization error when the scanner launches?](#why-do-i-get-a-resource-initialization-error-when-the-scanner-launches)
- [Why does `getDocumentImage(Opposite)` return `null`?](#why-does-getdocumentimageopposite-return-null)

**Result handling**

- [How do I use the scanned result?](#how-do-i-use-the-scanned-result)
- [What's the difference between the original image, the document image, and the portrait image?](#whats-the-difference-between-the-original-image-the-document-image-and-the-portrait-image)
- [How do I tell whether the user cancelled or no MRZ was found?](#how-do-i-tell-whether-the-user-cancelled-or-no-mrz-was-found)

**Integration and deployment**

- [Can I host the SDK assets somewhere other than the project's `public/` directory?](#can-i-host-the-sdk-assets-somewhere-other-than-the-projects-public-directory)
- [Do I need HTTPS to use the MRZ Scanner?](#do-i-need-https-to-use-the-mrz-scanner)

**Customization**

- [Can I localize or rebrand the on-screen messages and toolbar buttons?](#can-i-localize-or-rebrand-the-on-screen-messages-and-toolbar-buttons)
- [Can I scan the full camera frame instead of just the guide region?](#can-i-scan-the-full-camera-frame-instead-of-just-the-guide-region)

## Capability and supported formats

### Can the MRZ Scanner process static images, or does it only work with live cameras?

Yes. In addition to scanning from a live camera, the MRZ Scanner reads static images and image files by passing them directly to [`launch(file)`]({{ site.api }}mrz-scanner.html#launch). PDFs require a small extra step; see [Can the MRZ Scanner read PDFs?](#can-the-mrz-scanner-read-pdfs).

### Can the MRZ Scanner read PDFs?

Yes, but PDFs are not accepted natively because the scanner processes images, not document files. Two pieces of config are required: set [`scannerViewConfig.loadImageAcceptedTypes`]({{ site.api }}mrz-scanner.html#mrzscannerviewconfig) to include `application/pdf`, and provide a `loadImageFileConverter` callback that takes the PDF and returns an image `Blob`. The [Static Images and PDFs guide]({{ site.guides }}mrz-scanner-static-image.html) walks through a complete single-page implementation using PDF.js.

### Which document types and MRZ formats does the MRZ Scanner recognize?

The scanner recognizes five ICAO MRTD formats: TD3 passports, TD1 ID cards, TD2 ID cards (which includes the French national ID), and the two machine-readable visa formats MRVA (TD3-sized) and MRVB (TD2-sized). All five are enabled by default. To restrict recognition to a subset, set [`mrzFormatType`]({{ site.api }}mrz-scanner.html#mrzscannerconfig) in the constructor config to an array of [`EnumMRZDocumentType`]({{ site.api }}enums-mrz-scanner.html) values.

### Does the MRZ Scanner work on mobile browsers?

Yes. The scanner UI is responsive and designed for mobile capture. The requirement is the same as on desktop: a modern browser with `getUserMedia` support and a secure context (HTTPS, or `http://localhost` for development). Mobile Safari, Chrome for Android, and the standard in-app browsers on iOS and Android all work.

### Does the MRZ Scanner work with React, Vue, or Angular?

Yes. The SDK is framework-agnostic and ships ESM, CommonJS, and IIFE bundles along with TypeScript declarations, so it integrates with any modern web stack. Framework-specific examples are available in the [MRZ Scanner code gallery]({{ site.codegallery }}demo/index.html) and the [public sample repository](https://github.com/Dynamsoft/mrz-scanner-javascript).

## Privacy and data handling

### Does the MRZ Scanner perform data validation?

**No**. MRZ Scanner JavaScript Edition performs all image capture, image enhancement, and MRZ parsing on-device, to give you full control of your data. The MRZ Scanner processes the data on the client device without passing the data to external servers. Your application must implement data validation on the decoded MRZ data after reading with the MRZ Scanner if your use case requires data validation.

### Does the MRZ Scanner work offline, and does it send any data to Dynamsoft?

All image capture, enhancement, and MRZ parsing run on-device in WebAssembly. The only outbound network traffic the SDK generates is license validation against Dynamsoft's licensing service over HTTPS, which transmits the license token only. Document images, MRZ text, and parsed field data never leave the device. If your application needs to run entirely offline (including without license validation), contact the [Dynamsoft team](https://www.dynamsoft.com/company/contact/) about an offline license model.

## Defaults and gotchas

### Why does scanning a TD1 or TD2 ID card prompt the user to flip the document?

v4 enables multi-side scanning by default (`returnPortraitImage: true` on `MRZScannerConfig`). For TD1 and TD2 ID cards the portrait is on the opposite side from the MRZ, so the scanner captures the MRZ side first, pauses (3 seconds by default, controlled by `scannerViewConfig.flipDocumentTimeout`), and prompts the user to flip the document before capturing the portrait. To disable the flip flow and end the scan after the MRZ side, set `returnPortraitImage: false` in the constructor config. For passports the portrait is on the same side as the MRZ, so the second capture is skipped automatically.

### Why does the document type now return a code like CT_MRTD_TD3_PASSPORT instead of a readable label?

In v4, `result.data.documentType` returns the underlying DCV code-type identifier (an `EnumCodeType` value like `"CT_MRTD_TD3_PASSPORT"` or `"CT_MRTD_TD1_ID"`), not the humanized label v3.x produced (`"Passport (TD3)"`). The change makes the value stable and machine-readable, but it does mean you can no longer display `documentType` directly to end users. Map the code-type to a localized label in your application code; the `MRZDataLabel` helper covers field *keys* only, not document-type values. See the [upgrade guide]({{ site.guides }}mrz-scanner-upgrade-guide.html#mrzdatadocumenttype-silently-changed-shape) for the full value table.

### Why do I get a resource-initialization error when the scanner launches?

This almost always means the SDK cannot find its engine assets. For npm-based installs, copy the three packages (`dynamsoft-mrz-scanner`, `dynamsoft-capture-vision-bundle`, `dynamsoft-capture-vision-data`) into your project's `public/` directory so they are served from `/`; the SDK's defaults locate every resource and you can omit `engineResourcePaths` entirely. If you need to host the assets elsewhere, set `engineResourcePaths` to `{ dcvBundle, dcvData }` URLs that point at each package's contents. See the [User Guide]({{ site.guides }}mrz-scanner.html) for the full resource hosting setup.

### Why does `getDocumentImage(Opposite)` return `null`?

`Opposite` is only populated when multi-side scanning ran, which happens for TD1 and TD2 ID cards with `returnPortraitImage: true` (the v4 default). For passports the portrait is on the same side as the MRZ, so `Opposite` always returns `null`. Disabling the flip flow with `returnPortraitImage: false` also means every `Opposite` getter call returns `null`, because the opposite side was never captured. Same rule applies to `getOriginalImage(Opposite)` when `returnOriginalImage: true`.

## Result handling

### How do I use the scanned result?

The simplest implementation uses the default UI and does not take action after scanning. You can customize the behavior by handling the result of the [`launch()`]({{ site.api }}mrz-scanner.html#launch) call, which returns a [`MRZResult`]({{ site.api }}mrz-scanner.html#mrzresult). The result carries the parsed MRZ data on `result.data` (separated into fields), the captured images via the [`getOriginalImage`]({{ site.api }}mrz-scanner.html#getoriginalimage), [`getDocumentImage`]({{ site.api }}mrz-scanner.html#getdocumentimage), and [`getPortraitImage`]({{ site.api }}mrz-scanner.html#getportraitimage) getters, and a `result.status` field that distinguishes success from cancellation.

### What's the difference between the original image, the document image, and the portrait image?

The result exposes three image getters, each returning a different representation of the captured frame:

- **`result.getOriginalImage(side)`** returns the full unmodified camera frame. Off by default; set `returnOriginalImage: true` on the constructor config to receive it.
- **`result.getDocumentImage(side)`** returns the deskewed and cropped document. On by default (`returnDocumentImage: true`). This is the closest equivalent to a "scanned page" image.
- **`result.getPortraitImage()`** returns the photo crop. No `side` argument because each scan produces at most one portrait. On by default (`returnPortraitImage: true`), and this flag also gates whether multi-side scanning runs.

Each getter returns `null` when the corresponding image was not requested via the `return*Image` flags or could not be captured.

### How do I tell whether the user cancelled or no MRZ was found?

v4 collapses both cases into a missing `data` field, so the canonical check is `if (!result?.data)`. If you need to distinguish them, inspect `result.status`, which is now an `EnumResultStatus` value directly (not a wrapper object as it was in v3.x): `RS_CANCELLED` indicates user cancellation, `RS_FAILED` indicates no MRZ was detected, and `RS_SUCCESS` means `data` is present.

## Integration and deployment

### Can I host the SDK assets somewhere other than the project's `public/` directory?

Yes. Stage the contents of the `dynamsoft-capture-vision-bundle` and `dynamsoft-capture-vision-data` npm packages anywhere you control (a CDN, S3, an internal mirror), then set `engineResourcePaths: { dcvBundle: "...", dcvData: "..." }` in the constructor config. The two paths must point at the directory containing each package's contents. Cross-origin works as long as the host serves the WebAssembly file with the correct MIME type and CORS headers.

### Do I need HTTPS to use the MRZ Scanner?

In production, yes. Camera access through `getUserMedia` requires a secure context, and license validation goes over HTTPS. For local development, browsers exempt `http://localhost` (and on Chromium, `file:///`) from the secure-context requirement, so a plain `http://localhost:3000` dev server works without further setup. To test from a phone on the same LAN you will need a dev server that supports HTTPS; the [User Guide]({{ site.guides }}mrz-scanner.html) shows a Vite-based setup using `@vitejs/plugin-basic-ssl`.

## Customization

### Can I localize or rebrand the on-screen messages and toolbar buttons?

Yes, and from JavaScript without forking any assets. The [`scannerViewConfig`]({{ site.api }}mrz-scanner.html#mrzscannerviewconfig) object accepts four customization interfaces:

- **`messagesConfig`**: every on-screen prompt and error string (including the `{seconds}` placeholder used in the flip-document countdown).
- **`toolbarButtonsConfig`**: per-button icon, label, and visibility overrides for the seven scanner toolbar buttons.
- **`formatSelectorConfig`**: labels for the four format selector buttons.
- **`themeConfig`**: color, typography, and spacing tokens for the scanner overlay.

See the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html) for the complete option tables.

### Can I scan the full camera frame instead of just the guide region?

Yes. Set `scannerViewConfig.enableScanRegion: false`. The on-screen guide overlay disappears and the scanner reads the entire camera frame, which can help if your users frame documents loosely. The tradeoff is that a wider read region picks up incidental text in the background and slightly slows recognition.
