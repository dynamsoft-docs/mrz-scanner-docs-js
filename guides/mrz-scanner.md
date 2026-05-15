---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition
keywords: Documentation, MRZ Scanner, Dynamsoft MRZ Scanner JavaScript Edition,
description: Dynamsoft MRZ Scanner User Guide
permalink: /guides/mrz-scanner.html
---

# User Guide for the MRZ Scanner JavaScript Edition

This user guide demonstrates how to integrate the MRZ Scanner JavaScript Edition SDK into a production web application. You'll install the library via npm, build a scanner that supports both live-camera and file-upload entry points, render the parsed MRZ data alongside the cropped document and portrait images, and deploy the application from your own server.

> [!TIP]
> If you're just trying out the MRZ Scanner for the first time and want a single-file Hello World you can open from disk, see the [Quick Start]({{ site.guides }}mrz-scanner-quick-start.html) instead.

## License

### Trial License

Get started with a free 30-day trial license:

{% include trialLicense.html %}

The trial license can be renewed twice via the [customer portal](https://www.dynamsoft.com/customer/license/trialLicense/?product=mrz&utm_source=guide&package=js) (15 days each renewal), giving you 60 days total for development and evaluation. Contact the [Dynamsoft Support Team](https://www.dynamsoft.com/company/contact/) if you need additional time or have questions.

> [!NOTE]
> The **MRZ Scanner** license includes licenses for **Dynamsoft Label Recognizer**, **Dynamsoft Code Parser**, and **Dynamsoft Camera Enhancer**, as the MRZ Scanner is built on these three products.

### Full License

If you are fully satisfied with the solution and would like to move forward with a full license, please contact the [Dynamsoft Sales Team](https://www.dynamsoft.com/company/contact/).

## Prerequisites

> [!TIP]
> Visit the [Introduction]({{ site.introduction }}index.html) page to learn about MRZ document formats, the MRZ Scanner architecture, and system requirements.

You'll need:

- **Node.js** `^24.0.0` and **npm** `^11.0.0` to install the package and run the development server.
- A **trial or full license key** (see [License](#license)).
- A modern browser with `getUserMedia` support for camera scanning.

## Including the Library

You can include the MRZ Scanner SDK in your application by installing it from npm or by building it from the source repository.

<div class="multi-panel-switching-prefix"></div>

<div class="multi-panel-start"></div>
<div class="multi-panel-title">Install via npm</div>

The recommended way to include the SDK in a production application is to install it as a project dependency:

```sh
npm i dynamsoft-mrz-scanner@4.0.0 -E
# or
yarn add dynamsoft-mrz-scanner@4.0.0 -E
```

This installs the MRZ Scanner package along with its two peer dependencies, **`dynamsoft-capture-vision-bundle`** and **`dynamsoft-capture-vision-data`**. Together they provide the JavaScript bundle, the WebAssembly engine, and the model and template data files that power MRZ recognition.

After installation, the layout under `node_modules/` looks like:

```
node_modules/
├── dynamsoft-mrz-scanner/              # the library bundle and UI/template assets
├── dynamsoft-capture-vision-bundle/    # DCV engine (JS + WASM)
└── dynamsoft-capture-vision-data/      # DCV model, template, and parser data
```

> [!WARNING]
> When installing via npm, you must tell the SDK where to load the DCV engine resources from by setting the `engineResourcePaths` configuration option. The walkthrough in the next section shows how. See the [MRZScannerConfig API]({{ site.api }}mrz-scanner.html#mrzscannerconfig) for the full option reference.

<div class="multi-panel-end"></div>

<div class="multi-panel-start"></div>
<div class="multi-panel-title">Build from Source</div>

For deeper customization or to track the latest changes ahead of an npm release, you can build the SDK from the source repository.

The MRZ Scanner is built on three Dynamsoft products: [**Dynamsoft Label Recognizer**]({{ site.dlr_js }}api-reference/label-recognizer-module.html?lang=javascript), [**Dynamsoft Code Parser**]({{ site.dcp_js }}api-reference/code-parser-module.html?lang=javascript), and [**Dynamsoft Camera Enhancer**]({{ site.dce_js }}api-reference/index.html?lang=javascript). Building from source gives you direct access to the views and helpers that wire those products together.

Follow these steps:

1. Clone the [GitHub repository](https://github.com/Dynamsoft/mrz-scanner-javascript) (or download a ZIP and extract it).

2. From the project root, install dependencies:

    ```bash
    npm install
    ```

3. Build the library bundles:

    ```bash
    npm run build
    ```

    This produces the `dist/` folder containing everything you need to include in your own application: `dist/mrz-scanner.mjs` (ESM), `dist/mrz-scanner.cjs` (CommonJS), `dist/mrz-scanner.bundle.js` (IIFE), and the type declarations in `dist/mrz-scanner.d.ts`. Reference these directly from your project or publish them to your own registry.

4. **(Optional)** To try out the samples that ship with the repository, start the bundled Vite dev server:

    ```bash
    npm run dev
    ```

    The dev server serves the `samples/` folder over HTTPS with a self-signed certificate, using the `dist/` you just built in step 3. Open the URL printed in the terminal output and navigate to a sample (for example, `samples/demo/index.html`). This step is not required to produce or use the built library — `dist/` already contains everything your own application needs.

<div class="multi-panel-end"></div>

<div class="multi-panel-switching-end"></div>

## Building a Production Sample

This section walks you through building a complete MRZ Scanner integration end to end: a home screen with camera-scan and file-upload entry points, a result view that displays the portrait, the processed and original document images, and the parsed MRZ fields, and a re-scan flow that returns the user to a fresh scanning session.

The walkthrough focuses on the SDK API and integration patterns rather than visual design — styling itself isn't discussed in the steps below. The HTML and JS, however, use class names that line up with the official Dynamsoft demo's stylesheets, so dropping the demo's CSS files into your project gives you a fully styled, responsive result view with no markup changes. See the [Reference](#reference-the-dynamsoft-mrz-scanner-demo) at the end of this section for the two files to lift.

This walkthrough assumes you've installed the SDK via npm in a project at `mrz-scanner-app/` and that the page is served from the project root, so paths into `node_modules/` resolve as written. Adapt the paths to match your project layout.

If you don't already have a project set up, create one now:

```bash
mkdir mrz-scanner-app
cd mrz-scanner-app
npm init -y
npm i dynamsoft-mrz-scanner@4.0.0 -E
```

This produces a `package.json` and installs the SDK along with its two peer dependencies under `node_modules/`. The rest of the walkthrough adds an `index.html` and a `<script type="module">` block to this project.

### Step 1: Set Up the App Structure

Create a file named `index.html` at the project root with the following structure. The page has two top-level sections — `#home-section` (start screen) and `#result-section` (result view) — and the SDK bundle is loaded from `node_modules/`.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner</title>
    <script src="node_modules/dynamsoft-mrz-scanner/dist/mrz-scanner.bundle.js"></script>
    <style>
      /* Minimal home-section styling so the entry-point buttons remain
         visible whether or not the demo's stylesheets are linked. */
      #home-section { padding: 24px; display: flex; flex-direction: column; gap: 12px; align-items: flex-start; }
      #home-section button { padding: 12px 16px; color: #fff; background: #333; }
    </style>
  </head>

  <body data-view="landing">
    <!-- Home: start screen with the two scan entry points -->
    <div id="home-section">
      <h1>MRZ Scanner</h1>
      <button id="startScan">Start Camera Scan</button>
      <button id="uploadFile">Scan from File</button>
      <div id="home-error"></div>
    </div>

    <!-- Result: rendered after a successful scan -->
    <div id="result-section" style="display: none">
      <div class="result-header">
        <h2>Result</h2>
      </div>

      <!-- Left panel: summary + image groups -->
      <div class="result-left">
        <div class="summary-card">
          <div class="summary-info">
            <div class="person-name" id="res-name"></div>
            <div class="person-details" id="res-gender-age"></div>
            <div class="person-details" id="res-expiry"></div>
          </div>
          <div class="portrait-box" id="portrait-container"></div>
        </div>

        <div class="data-section">
          <div class="data-section-title">Processed images</div>
          <div id="processed-images"></div>
        </div>

        <div class="data-section">
          <div class="data-section-title">Original images</div>
          <div id="original-images"></div>
        </div>
      </div>

      <!-- Right panel: MRZ data fields + raw text -->
      <div class="result-right">
        <div class="data-section">
          <div class="data-section-title">Personal Info</div>
          <div id="personal-info-rows"></div>
        </div>
        <div class="data-section">
          <div class="data-section-title">Document Info</div>
          <div id="document-info-rows"></div>
        </div>
        <div class="mrz-text-section">
          <div class="data-section-title">Raw MRZ Text</div>
          <div class="mrz-raw-text" id="mrz-raw-text"></div>
        </div>
      </div>

      <!-- Action buttons -->
      <div class="action-buttons">
        <button type="button" class="btn-rescan">Re-scan</button>
        <button type="button" class="btn-home">Return home</button>
      </div>
    </div>

    <script type="module">
      // The application code from the next steps goes here.
    </script>
  </body>
</html>
```

Once `mrz-scanner.bundle.js` is loaded, it exposes a global `Dynamsoft` namespace that holds the `MRZScanner` constructor, the `EnumDocumentSide` enum, and other helpers used below.

> [!NOTE]
> If you're using a bundler (Vite, Webpack, Rollup, etc.) instead of a `<script>` tag, import the SDK from the `dynamsoft-mrz-scanner` package and use the named exports (`MRZScanner`, `EnumDocumentSide`, etc.) directly. The configuration shown below is identical.

### Step 2: Initialize the Scanner

Inside the `<script type="module">` block, declare a module-level `mrzScanner` variable and create the initial instance:

```js
let mrzScanner;

try {
  mrzScanner = new Dynamsoft.MRZScanner({
    license: "YOUR_LICENSE_KEY_HERE",
    engineResourcePaths: {
      dcvBundle: "node_modules/dynamsoft-capture-vision-bundle/dist/",
      dcvData: "node_modules/dynamsoft-capture-vision-data/",
    },
    returnOriginalImage: true,
    returnDocumentImage: true,
    returnPortraitImage: true,
  });
} catch (error) {
  document.getElementById("home-error").textContent =
    `Failed to initialize scanner: ${error.message}`;
  document.getElementById("startScan").disabled = true;
  document.getElementById("uploadFile").disabled = true;
}
```

What's going on here:

- **`license`** — replace `YOUR_LICENSE_KEY_HERE` with your trial or full key (see [License](#license)). An invalid license causes a launch error.
- **`engineResourcePaths`** — required for npm-installed projects. The bundle defaults to looking for DCV resources next to itself inside `dist/`, but the actual peer dependencies sit at the sibling locations shown above. Without this override, the WebAssembly and data files will return a 404.
- **`returnOriginalImage` / `returnDocumentImage` / `returnPortraitImage`** — control which image artifacts are attached to the result. `returnDocumentImage` and `returnPortraitImage` default to `true`; `returnOriginalImage` defaults to `false`. All three are set explicitly here so the result view can render every image kind the API offers. Setting `returnPortraitImage: false` also disables multi-side scanning — see [Multi-Side Scanning](#multi-side-scanning) below.

For the full list of configuration options, see the [MRZScannerConfig API]({{ site.api }}mrz-scanner.html#mrzscannerconfig).

### Step 3: Wire the Camera Scan Entry Point

Add a `startScanning` function that disables both entry-point buttons during the scan, calls `mrzScanner.launch()` to open the live-camera UI, and hands the result off to a `displayResults` function (defined in step 5):

```js
async function startScanning() {
  const startBtn = document.getElementById("startScan");
  const uploadBtn = document.getElementById("uploadFile");
  const homeError = document.getElementById("home-error");

  startBtn.disabled = true;
  uploadBtn.disabled = true;
  homeError.textContent = "";

  try {
    const result = await mrzScanner.launch();
    displayResults(result);
  } catch (error) {
    homeError.textContent = `Scanning error: ${error.message}`;
  } finally {
    startBtn.disabled = false;
    uploadBtn.disabled = false;
  }
}

document.getElementById("startScan").addEventListener("click", startScanning);
```

`launch()` opens the **MRZScannerView** — a full-screen container with a live camera feed, a guide frame, format selector, and toolbar buttons. When an MRZ is recognized, the promise resolves with an [**`MRZResult`**]({{ site.api }}mrz-scanner.html#mrzresult). When the user closes the scanner without scanning, the promise still resolves but with `result.data` undefined, which the `displayResults` function handles in step 5.

### Step 4: Wire the File-Upload Entry Point

`launch()` accepts an optional `Blob`, `File`, image URL, or HTML media element. When called with one of these, the scanner skips the camera and processes the supplied image directly. Wire up the upload button to a hidden `<input type="file">`, await the user's selection, and pass the resulting `File` to `launch()`:

```js
async function startFileUpload() {
  const startBtn = document.getElementById("startScan");
  const uploadBtn = document.getElementById("uploadFile");
  const homeError = document.getElementById("home-error");

  // Prompt the user to pick a file before doing any work
  const input = document.createElement("input");
  input.type = "file";
  input.accept = "image/*";

  const file = await new Promise((resolve) => {
    input.onchange = (e) => resolve(e.target.files?.[0] ?? null);
    input.addEventListener("cancel", () => resolve(null));
    input.click();
  });

  if (!file) return;

  startBtn.disabled = true;
  uploadBtn.disabled = true;
  homeError.textContent = "";

  try {
    const result = await mrzScanner.launch(file);
    displayResults(result);
  } catch (error) {
    homeError.textContent = `File scan error: ${error.message}`;
  } finally {
    startBtn.disabled = false;
    uploadBtn.disabled = false;
  }
}

document.getElementById("uploadFile").addEventListener("click", startFileUpload);
```

> [!TIP]
> To accept additional file types like PDF, set `input.accept` accordingly and convert the file to an image `Blob` before passing it to `launch()`. See [Setting up the MRZ Scanner for Static Images and PDFs]({{ site.guides }}mrz-scanner-static-image.html) for a complete PDF flow.

### Step 5: Render the Result

The `displayResults` function takes the `MRZResult` returned by `launch()` and populates the result view. It handles three concerns:

1. **Parsed data** — fields read from `result.data`.
2. **Images** — read from `result.getDocumentImage(side)`, `result.getOriginalImage(side)`, and `result.getPortraitImage()`. Each returns a `DSImageData` whose `toCanvas()` method produces an `HTMLCanvasElement` ready to append to the DOM. Each method also returns `null` when the matching `return*` config flag was disabled or when the side wasn't captured (see [Multi-Side Scanning](#multi-side-scanning) below for when `EnumDocumentSide.Opposite` is populated).
3. **View-state toggles** — hiding the home section and showing the result section.

Add the following helpers and the main `displayResults` function:

```js
// ─── Helpers ─────────────────────────────────────────────────────────────

function formatDate(dateObj) {
  if (!dateObj || !dateObj.year) return "—";
  const pad = (n) => String(n).padStart(2, "0");
  return `${dateObj.year}-${pad(dateObj.month)}-${pad(dateObj.day)}`;
}

function escapeHTML(str) {
  return String(str ?? "")
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;");
}

function dataRow(label, value) {
  return `<div class="data-row">
    <span class="data-row-label">${escapeHTML(label)}</span>
    <span class="data-row-value">${escapeHTML(value || "—")}</span>
  </div>`;
}

function appendImageCard(container, imageData) {
  if (!imageData?.toCanvas) return;
  const card = document.createElement("div");
  card.className = "tab-image-card";
  card.appendChild(imageData.toCanvas());
  container.appendChild(card);
}

function showHome() {
  document.body.dataset.view = "landing";
  document.getElementById("home-section").style.display = "";
  document.getElementById("result-section").style.display = "none";
}

function showResult() {
  document.body.dataset.view = "result";
  document.getElementById("home-section").style.display = "none";
  document.getElementById("result-section").style.display = "";
}

// ─── Main result-rendering function ──────────────────────────────────────

function displayResults(result) {
  // Cancelled scans and scans with no detected MRZ resolve with no `data`
  if (!result?.data) {
    document.getElementById("home-error").textContent =
      "No MRZ data detected. Please try again.";
    return;
  }

  const d = result.data;

  // --- Summary ---
  const name = [d.firstName, d.lastName].filter(Boolean).join(" ");
  document.getElementById("res-name").textContent = name || "—";

  const genderAge = [d.sex, d.age != null ? `${d.age} years old` : ""].filter(Boolean).join(", ");
  document.getElementById("res-gender-age").textContent = genderAge;

  const expiryStr = formatDate(d.dateOfExpiry);
  document.getElementById("res-expiry").textContent =
    expiryStr !== "—" ? `Expiry: ${expiryStr}` : "";

  // --- Portrait ---
  const portraitContainer = document.getElementById("portrait-container");
  portraitContainer.innerHTML = "";
  const portraitImage = result.getPortraitImage();
  if (portraitImage?.toCanvas) {
    portraitContainer.appendChild(portraitImage.toCanvas());
  }

  // --- Processed images: deskewed document crops via getDocumentImage() ---
  const processedPanel = document.getElementById("processed-images");
  processedPanel.innerHTML = "";
  appendImageCard(processedPanel, result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ));
  appendImageCard(processedPanel, result.getDocumentImage(Dynamsoft.EnumDocumentSide.Opposite));
  if (processedPanel.children.length === 0) {
    processedPanel.innerHTML = `<p class="tab-empty-msg">No document images available</p>`;
  }

  // --- Original images: raw captured frames via getOriginalImage() ---
  const originalPanel = document.getElementById("original-images");
  originalPanel.innerHTML = "";
  appendImageCard(originalPanel, result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ));
  appendImageCard(originalPanel, result.getOriginalImage(Dynamsoft.EnumDocumentSide.Opposite));
  if (originalPanel.children.length === 0) {
    originalPanel.innerHTML = `<p class="tab-empty-msg">No original images available</p>`;
  }

  // --- Parsed data ---
  document.getElementById("personal-info-rows").innerHTML =
    dataRow("Given Name", d.firstName) +
    dataRow("Surname", d.lastName) +
    dataRow("Date of Birth", formatDate(d.dateOfBirth)) +
    dataRow("Gender", d.sex) +
    dataRow("Nationality", d.nationality);

  document.getElementById("document-info-rows").innerHTML =
    dataRow("Doc. Type", d.documentType) +
    dataRow("Doc. Number", d.documentNumber) +
    dataRow("Issuing State", d.issuingState) +
    dataRow("Expiry Date", formatDate(d.dateOfExpiry));

  document.getElementById("mrz-raw-text").textContent = d.mrzText || "";

  // Reveal the result view
  showResult();
}
```

Key APIs in use:

- **`result.data`** — the parsed MRZ payload: `firstName`, `lastName`, `sex`, `age`, `nationality`, `documentType`, `documentNumber`, `issuingState`, `dateOfBirth`, `dateOfExpiry`, `mrzText`, and `optionalData1` / `optionalData2` when present. Dates are returned as `{ year, month, day }` objects.
- **`result.getDocumentImage(side)`** — deskewed crop of the document for the given side. `Dynamsoft.EnumDocumentSide.MRZ` is the side carrying the MRZ; `EnumDocumentSide.Opposite` is the opposite side, populated only when multi-side scanning runs (see [Multi-Side Scanning](#multi-side-scanning)).
- **`result.getOriginalImage(side)`** — the full unmodified frame for the given side. Only populated when `returnOriginalImage: true`.
- **`result.getPortraitImage()`** — the portrait crop, regardless of which side it was found on.
- **`DSImageData.toCanvas()`** — converts the image into an `HTMLCanvasElement` ready to append to the DOM. `toBlob()` is also available if you'd rather upload or store the image.

### Step 6: Wire Re-Scan and Return Home

After a result is rendered, the user can re-launch the scanner with the same configuration or return to the home screen. Add the wiring:

```js
async function rescan() {
  const homeError = document.getElementById("home-error");
  homeError.textContent = "";

  try {
    // launch() can be called repeatedly on the same instance — the SDK
    // automatically disposes and re-initializes between calls.
    const result = await mrzScanner.launch();
    displayResults(result);
  } catch (error) {
    showHome();
    homeError.textContent = `Scanning error: ${error.message}`;
  }
}

function returnHome() {
  showHome();
}

document.querySelector(".btn-rescan").addEventListener("click", rescan);
document.querySelector(".btn-home").addEventListener("click", returnHome);
```

> [!NOTE]
> If you need to **change the configuration** (different `mrzFormatType`, different `return*Image` flags, different `scannerViewConfig`) between scans, replace the instance instead of reusing it: `mrzScanner = new Dynamsoft.MRZScanner({ ...newConfig })` and then call `launch()`. Configuration is fixed at construction time. See [Lifecycle and Disposal](#lifecycle-and-disposal) for the full set of patterns.

### Putting It All Together

The full `index.html` after all six steps is shown below. Drop in your license key, serve the file (see [Local HTTPS via Vite](#local-https-via-vite) for a one-command dev server), and you have a working MRZ Scanner integration with camera and file-upload entry points, a result view, and a re-scan flow — all in one file.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner</title>
    <script src="node_modules/dynamsoft-mrz-scanner/dist/mrz-scanner.bundle.js"></script>
    <style>
      /* Minimal home-section styling so the entry-point buttons remain
         visible whether or not the demo's stylesheets are linked. */
      #home-section { padding: 24px; display: flex; flex-direction: column; gap: 12px; align-items: flex-start; }
      #home-section button { padding: 12px 16px; color: #fff; background: #333; }
    </style>
  </head>

  <body data-view="landing">
    <!-- Home: start screen with the two scan entry points -->
    <div id="home-section">
      <h1>MRZ Scanner</h1>
      <button id="startScan">Start Camera Scan</button>
      <button id="uploadFile">Scan from File</button>
      <div id="home-error"></div>
    </div>

    <!-- Result: rendered after a successful scan -->
    <div id="result-section" style="display: none">
      <div class="result-header">
        <h2>Result</h2>
      </div>

      <!-- Left panel: summary + image groups -->
      <div class="result-left">
        <div class="summary-card">
          <div class="summary-info">
            <div class="person-name" id="res-name"></div>
            <div class="person-details" id="res-gender-age"></div>
            <div class="person-details" id="res-expiry"></div>
          </div>
          <div class="portrait-box" id="portrait-container"></div>
        </div>

        <div class="data-section">
          <div class="data-section-title">Processed images</div>
          <div id="processed-images"></div>
        </div>

        <div class="data-section">
          <div class="data-section-title">Original images</div>
          <div id="original-images"></div>
        </div>
      </div>

      <!-- Right panel: MRZ data fields + raw text -->
      <div class="result-right">
        <div class="data-section">
          <div class="data-section-title">Personal Info</div>
          <div id="personal-info-rows"></div>
        </div>
        <div class="data-section">
          <div class="data-section-title">Document Info</div>
          <div id="document-info-rows"></div>
        </div>
        <div class="mrz-text-section">
          <div class="data-section-title">Raw MRZ Text</div>
          <div class="mrz-raw-text" id="mrz-raw-text"></div>
        </div>
      </div>

      <!-- Action buttons -->
      <div class="action-buttons">
        <button type="button" class="btn-rescan">Re-scan</button>
        <button type="button" class="btn-home">Return home</button>
      </div>
    </div>

    <script type="module">
      // ─── Initialize the scanner ──────────────────────────────────────────
      let mrzScanner;

      try {
        mrzScanner = new Dynamsoft.MRZScanner({
          license: "YOUR_LICENSE_KEY_HERE",
          engineResourcePaths: {
            dcvBundle: "node_modules/dynamsoft-capture-vision-bundle/dist/",
            dcvData: "node_modules/dynamsoft-capture-vision-data/",
          },
          returnOriginalImage: true,
          returnDocumentImage: true,
          returnPortraitImage: true,
        });
      } catch (error) {
        document.getElementById("home-error").textContent =
          `Failed to initialize scanner: ${error.message}`;
        document.getElementById("startScan").disabled = true;
        document.getElementById("uploadFile").disabled = true;
      }

      // ─── Helpers ─────────────────────────────────────────────────────────

      function formatDate(dateObj) {
        if (!dateObj || !dateObj.year) return "—";
        const pad = (n) => String(n).padStart(2, "0");
        return `${dateObj.year}-${pad(dateObj.month)}-${pad(dateObj.day)}`;
      }

      function escapeHTML(str) {
        return String(str ?? "")
          .replace(/&/g, "&amp;")
          .replace(/</g, "&lt;")
          .replace(/>/g, "&gt;");
      }

      function dataRow(label, value) {
        return `<div class="data-row">
          <span class="data-row-label">${escapeHTML(label)}</span>
          <span class="data-row-value">${escapeHTML(value || "—")}</span>
        </div>`;
      }

      function appendImageCard(container, imageData) {
        if (!imageData?.toCanvas) return;
        const card = document.createElement("div");
        card.className = "tab-image-card";
        card.appendChild(imageData.toCanvas());
        container.appendChild(card);
      }

      function showHome() {
        document.body.dataset.view = "landing";
        document.getElementById("home-section").style.display = "";
        document.getElementById("result-section").style.display = "none";
      }

      function showResult() {
        document.body.dataset.view = "result";
        document.getElementById("home-section").style.display = "none";
        document.getElementById("result-section").style.display = "";
      }

      // ─── Entry points ────────────────────────────────────────────────────

      async function startScanning() {
        const startBtn = document.getElementById("startScan");
        const uploadBtn = document.getElementById("uploadFile");
        const homeError = document.getElementById("home-error");

        startBtn.disabled = true;
        uploadBtn.disabled = true;
        homeError.textContent = "";

        try {
          const result = await mrzScanner.launch();
          displayResults(result);
        } catch (error) {
          homeError.textContent = `Scanning error: ${error.message}`;
        } finally {
          startBtn.disabled = false;
          uploadBtn.disabled = false;
        }
      }

      async function startFileUpload() {
        const startBtn = document.getElementById("startScan");
        const uploadBtn = document.getElementById("uploadFile");
        const homeError = document.getElementById("home-error");

        const input = document.createElement("input");
        input.type = "file";
        input.accept = "image/*";

        const file = await new Promise((resolve) => {
          input.onchange = (e) => resolve(e.target.files?.[0] ?? null);
          input.addEventListener("cancel", () => resolve(null));
          input.click();
        });

        if (!file) return;

        startBtn.disabled = true;
        uploadBtn.disabled = true;
        homeError.textContent = "";

        try {
          const result = await mrzScanner.launch(file);
          displayResults(result);
        } catch (error) {
          homeError.textContent = `File scan error: ${error.message}`;
        } finally {
          startBtn.disabled = false;
          uploadBtn.disabled = false;
        }
      }

      // ─── Result rendering ────────────────────────────────────────────────

      function displayResults(result) {
        if (!result?.data) {
          document.getElementById("home-error").textContent =
            "No MRZ data detected. Please try again.";
          return;
        }

        const d = result.data;

        // Summary
        const name = [d.firstName, d.lastName].filter(Boolean).join(" ");
        document.getElementById("res-name").textContent = name || "—";

        const genderAge = [d.sex, d.age != null ? `${d.age} years old` : ""].filter(Boolean).join(", ");
        document.getElementById("res-gender-age").textContent = genderAge;

        const expiryStr = formatDate(d.dateOfExpiry);
        document.getElementById("res-expiry").textContent =
          expiryStr !== "—" ? `Expiry: ${expiryStr}` : "";

        // Portrait
        const portraitContainer = document.getElementById("portrait-container");
        portraitContainer.innerHTML = "";
        const portraitImage = result.getPortraitImage();
        if (portraitImage?.toCanvas) {
          portraitContainer.appendChild(portraitImage.toCanvas());
        }

        // Processed images
        const processedPanel = document.getElementById("processed-images");
        processedPanel.innerHTML = "";
        appendImageCard(processedPanel, result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ));
        appendImageCard(processedPanel, result.getDocumentImage(Dynamsoft.EnumDocumentSide.Opposite));
        if (processedPanel.children.length === 0) {
          processedPanel.innerHTML = `<p class="tab-empty-msg">No document images available</p>`;
        }

        // Original images
        const originalPanel = document.getElementById("original-images");
        originalPanel.innerHTML = "";
        appendImageCard(originalPanel, result.getOriginalImage(Dynamsoft.EnumDocumentSide.MRZ));
        appendImageCard(originalPanel, result.getOriginalImage(Dynamsoft.EnumDocumentSide.Opposite));
        if (originalPanel.children.length === 0) {
          originalPanel.innerHTML = `<p class="tab-empty-msg">No original images available</p>`;
        }

        // Parsed data
        document.getElementById("personal-info-rows").innerHTML =
          dataRow("Given Name", d.firstName) +
          dataRow("Surname", d.lastName) +
          dataRow("Date of Birth", formatDate(d.dateOfBirth)) +
          dataRow("Gender", d.sex) +
          dataRow("Nationality", d.nationality);

        document.getElementById("document-info-rows").innerHTML =
          dataRow("Doc. Type", d.documentType) +
          dataRow("Doc. Number", d.documentNumber) +
          dataRow("Issuing State", d.issuingState) +
          dataRow("Expiry Date", formatDate(d.dateOfExpiry));

        document.getElementById("mrz-raw-text").textContent = d.mrzText || "";

        showResult();
      }

      // ─── Re-scan and return-home ─────────────────────────────────────────

      async function rescan() {
        const homeError = document.getElementById("home-error");
        homeError.textContent = "";

        try {
          const result = await mrzScanner.launch();
          displayResults(result);
        } catch (error) {
          showHome();
          homeError.textContent = `Scanning error: ${error.message}`;
        }
      }

      function returnHome() {
        showHome();
      }

      // ─── Wire event listeners ────────────────────────────────────────────

      document.getElementById("startScan").addEventListener("click", startScanning);
      document.getElementById("uploadFile").addEventListener("click", startFileUpload);
      document.querySelector(".btn-rescan").addEventListener("click", rescan);
      document.querySelector(".btn-home").addEventListener("click", returnHome);
    </script>
  </body>
</html>
```

### Reference: The Dynamsoft MRZ Scanner Demo

The walkthrough above gives you a working API integration with no styling applied. To skin it with the official Dynamsoft demo's dark, mobile-and-desktop-responsive result view, copy [`samples/demo/css/index.css`](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/css/index.css) and [`samples/demo/css/result.css`](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/css/result.css) into a `css/` folder in your project, and add two `<link>` tags to your `<head>`:

```html
<link rel="stylesheet" href="css/index.css" />
<link rel="stylesheet" href="css/result.css" />
```

That's the full styling integration — the walkthrough's markup and JS already use the class names those two stylesheets expect, so no further code changes are required.

For everything else the demo includes — a branded mobile landing page with a photo banner, a desktop-to-mobile QR-code handoff, an info-menu dropdown, and the responsive layouts that wrap them — see the full demo source at [`samples/demo/`](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/demo) in the [`Dynamsoft/mrz-scanner-javascript`](https://github.com/Dynamsoft/mrz-scanner-javascript) repository.

The demo is one example of how to wrap the scanner rather than a prescription. The MRZ Scanner API hands you a parsed `MRZData` object and one or more `DSImageData` objects per scan — how you frame and chrome the rest of the app is entirely up to your design system.

## Multi-Side Scanning

Multi-side scanning is what produces a populated `EnumDocumentSide.Opposite` image on the result. It is **on by default** because `returnPortraitImage` defaults to `true`; setting `returnPortraitImage: false` disables it and `getDocumentImage(Opposite)` / `getOriginalImage(Opposite)` will always return `null`.

The flow when multi-side scanning is enabled:

1. The user scans the **MRZ side**. This populates the primary images returned by `getOriginalImage(MRZ)` and `getDocumentImage(MRZ)`.
2. The scanner inspects the same side for a portrait. If it finds one (e.g. a passport, where the MRZ and the portrait are on the same photo page), the portrait crop fills `getPortraitImage()` and the scan ends. **`EnumDocumentSide.Opposite` stays `null`.**
3. If no portrait is found on the MRZ side (typical for TD1 / TD2 ID cards, where the portrait sits on the side opposite the MRZ), the UI prompts the user to flip the document after `flipDocumentTimeout` milliseconds (default `3000`), captures the other side, and populates `getOriginalImage(Opposite)` and `getDocumentImage(Opposite)` along with the portrait crop.

The API uses **`MRZ`** and **`Opposite`** rather than front/back because document layouts vary by country — there's no guarantee the MRZ is on the physical back or that the portrait is on the physical front. The scanner always captures the **MRZ side first** and labels it `MRZ`; whichever side the user flips to is labeled `Opposite`.

In practice:

| Document type | `getDocumentImage(MRZ)` | `getDocumentImage(Opposite)` | `getPortraitImage()` |
| --- | --- | --- | --- |
| Passports (TD3) | populated | `null` | populated (from the same side) |
| ID cards (TD1 / TD2) | populated | populated | populated (from the `Opposite` side) |
| Any document, with `returnPortraitImage: false` | populated | `null` | `null` |

The same matrix applies to `getOriginalImage(side)` when `returnOriginalImage: true`.

To customize the flip-document countdown duration, set `scannerViewConfig.flipDocumentTimeout` on the `MRZScannerConfig`. See the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html) for the full list of multi-side-scanning options.

## Lifecycle and Disposal

`MRZScanner` instances are stateful — they hold references to the camera, the WebAssembly engine, and DOM containers. The SDK manages most of this for you, but it's worth understanding the lifecycle when designing the re-scan flow or when integrating into a single-page application that mounts and unmounts the scanner.

The key behaviors:

- **`launch()` always disposes when it resolves.** On success, failure, or cancellation, `launch()` runs `dispose()` in its `finally` block. Camera handles, the CV router, and DOM containers are torn down, and the instance is marked uninitialized.
- **`launch()` is re-entrant.** Calling `launch()` again on the same instance re-initializes from scratch. You don't need to manually dispose between calls.
- **`dispose()` is idempotent.** Calling it twice is harmless; subsequent calls are no-ops.
- **Configuration is fixed at construction time.** To change `license`, `mrzFormatType`, the `return*Image` flags, `engineResourcePaths`, or `scannerViewConfig`, you must construct a new `MRZScanner` instance.

These behaviors collapse into three patterns:

| Scenario | Right pattern |
| --- | --- |
| Re-scan with the **same configuration** | Call `mrzScanner.launch()` again on the existing instance — no manual `dispose()`, no new instance. |
| Re-scan with a **different configuration** | Replace the instance: `mrzScanner = new Dynamsoft.MRZScanner({ ...newConfig })` and then `await mrzScanner.launch()`. The previous instance auto-disposed when its `launch()` resolved. |
| Tear down without launching again (SPA component unmount, navigating to a different view permanently) | Call `mrzScanner.dispose()` explicitly. Without this, no automatic cleanup runs. |

> [!NOTE]
> Calling `mrzScanner.dispose()` defensively before re-creating the instance is redundant — the previous instance was already disposed by its own `launch()` finally block — but it does no harm. The simpler pattern shown in the table above is canonical.

## Understanding the MRZScannerView

When `launch()` is called without an image source, the MRZ Scanner opens the **MRZScannerView** in a full-screen container. The view is configured via [**`MRZScannerViewConfig`**]({{ site.api }}mrz-scanner.html#mrzscannerviewconfig), which is nested under the `scannerViewConfig` field of `MRZScannerConfig`.

The view consists of these UI elements:

**Core Scanning Interface:**

1. **Camera View** — the camera viewfinder UI component that occupies the majority of the space, giving the user a clear view and precise control of the image being scanned.

2. **Scan Guide Frame** — an overlay that guides the user to position the MRZ document correctly for fast and accurate scanning. Enabled **by default**, and can be hidden via `MRZScannerViewConfig.showScanGuide`. When enabled, the scanner crops the region outside the guide frame.

    <div align="center">
       <img src="../assets/imgs/mrz-scan-guide-new.png" alt="Scan Guide Frames" width="60%" />
    </div><br />

3. **Format Selector** — allows the user to choose which MRZ formats to recognize. Available formats are configured via `MRZScannerConfig.mrzFormatType`, while visibility is controlled via `MRZScannerViewConfig.showFormatSelector`. To learn about MRZ formats, see the [Introduction]({{ site.introduction }}index.html#supported-mrz-formats) page.

    <div align="center">
       <img src="../assets/imgs/format-selector-new.jpg" alt="Format Selector" width="60%" />
    </div><br />

**Camera Controls:**

4. **Resolution / Camera Select Dropdown** — switch between available cameras or select different resolutions for the active camera.

5. **Flash Button** — toggles the camera flash when available. Only appears if the device and browser support camera flash.

**Additional Options:**

6. **Load Image Button** — scan an MRZ from an image file stored on the device. (Distinct from the application-level file-upload entry point in step 4 — this button is part of the scanner UI and works inside an active camera session.)

7. **Sound Button** — toggle audio feedback (beep) when an MRZ is recognized.

8. **Close Scanner Button** — closes the MRZ Scanner. The `launch()` promise resolves with no `data`, and your `displayResults` function should treat that as a cancellation.

> [!NOTE]
> To learn more about customizing the MRZScannerView and its UI elements — toolbar buttons, theme, on-screen messages, format selector labels, the multi-side scanning timeout, and more — see the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).

## Local HTTPS via Vite

The MRZ Scanner needs a secure context (HTTPS) for camera access and license validation. For development — especially when testing on a phone over the LAN — Vite combined with the [`@vitejs/plugin-basic-ssl`](https://www.npmjs.com/package/@vitejs/plugin-basic-ssl) plugin produces a self-signed certificate on the fly with no server setup.

Install Vite and the plugin in your project:

```bash
npm i -D vite @vitejs/plugin-basic-ssl
```

Create a `vite.config.ts` (or `vite.config.js`) at your project root:

```ts
import { defineConfig } from "vite";
import basicSsl from "@vitejs/plugin-basic-ssl";

export default defineConfig({
  plugins: [basicSsl()],
  server: {
    host: "0.0.0.0",
    headers: {
      // Enable SharedArrayBuffer so DCV can use the multi-threaded (pthread)
      // WASM variant for better performance.
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
});
```

Then run:

```bash
npx vite
```

Vite prints a `https://localhost:<port>` URL plus the LAN-accessible URL bound by `host: "0.0.0.0"`. Open the page, accept the self-signed certificate warning, and grant camera permission. The MRZ Scanner UI launches over HTTPS.

> [!NOTE]
> The COOP and COEP headers are not strictly required, but they enable `SharedArrayBuffer`, which lets the DCV engine load the multi-threaded WebAssembly variant for noticeably faster MRZ recognition. If your application embeds third-party content that doesn't set the matching `Cross-Origin-Resource-Policy` header, omit the COOP/COEP block.

> [!WARNING]
> Self-signed certificates trigger a "Your connection is not private" warning on first visit. Click **Advanced** → **Proceed to localhost (unsafe)** to continue. Browsers do not persist this exception across full reinstalls or profile resets.

When you're ready to ship the application to a real server, see [Deployment](#deployment) below for the production hosting requirements.

## Deployment

When you're ready to ship your application to a production server, the MRZ Scanner is a static web SDK — any server capable of serving HTML, JavaScript, WebAssembly, and JSON files works. The requirements below apply to that production server; for local development over HTTPS, see [Local HTTPS via Vite](#local-https-via-vite) above.

### Secure Context (HTTPS)

Serve your application over HTTPS in production. This is required because:

- **Camera Access** — browsers only grant access to the camera video stream in a secure context.
- **License Validation** — the Dynamsoft License requires a secure context to function.

> [!NOTE]
> For development, some browsers (like Chrome) allow camera access on `http://127.0.0.1`, `http://localhost`, or `file:///` URLs as a developer convenience.

### MIME Type for `.wasm` Files

Configure your server to send the correct `Content-Type: application/wasm` header for WebAssembly files. Configuration varies by server:

- [Apache](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Apache_Configuration_htaccess#media_types_and_character_encodings)
- [IIS](https://docs.microsoft.com/en-us/iis/configuration/system.webserver/staticcontent/mimemap)
- [NGINX](https://www.nginx.com/resources/wiki/start/topics/examples/full/#mime-types)

### Resource Hosting

After installing the SDK via npm, copy the contents of `node_modules/dynamsoft-mrz-scanner/dist/`, `node_modules/dynamsoft-capture-vision-bundle/dist/`, and `node_modules/dynamsoft-capture-vision-data/` into your deployment artifact, and update `engineResourcePaths` in your `MRZScannerConfig` to match the URLs your server serves them from.

## Next Steps

- [Customizing the MRZ Scanner]({{ site.guides }}mrz-scanner-customization.html) — tailor the scanner UI, toolbar buttons, theme, on-screen messages, and multi-side scanning behavior.
- [Setting up the MRZ Scanner for Static Images and PDFs]({{ site.guides }}mrz-scanner-static-image.html) — process image and PDF files as input instead of (or in addition to) the live camera feed.
- For framework-specific implementations (React, Angular, Vue), see the [framework samples]({{ site.codegallery }}index.html#frameworks).
