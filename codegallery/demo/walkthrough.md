---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - Demo Walkthrough
keywords: Documentation, MRZ Scanner JavaScript Edition, Demo, Walkthrough
breadcrumbText: Demo Walkthrough
description: Step-by-step walkthrough of a simpler MRZ Scanner demo implementation — home and result UI without the demo's branded landing page, QR-code handoff, or full theming.
permalink: /codegallery/demo/walkthrough.html
---

# MRZ Scanner Demo Walkthrough

This walkthrough breaks down a simpler version of the [MRZ Scanner Demo](index.html), with the same home and result UI but without the demo's branded landing page, desktop-to-mobile QR-code handoff, or theming layer. It's the easiest entry point if you want to mirror the demo's structure in your own project.

The walkthrough assumes you've already installed the SDK and have its runtime assets reachable from your page — see the [User Guide]({{ site.guides }}mrz-scanner.html) for the npm install and `public/`-folder setup.

> [!IMPORTANT]
> **To get a styled result view that looks like the demo**, copy [`samples/demo/css/index.css`](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/css/index.css) and [`samples/demo/css/result.css`](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/css/result.css) from the [demo source](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/demo) into a `css/` folder in your project before you start. The HTML below uses the class names those stylesheets expect and references them with `<link>` tags in `<head>`. **Skip this step and the result view will render unstyled.** The markup will still work, but it won't look like the demo.

## Step 1: Set Up the App Structure

Create a file named `index.html` at the project root with the following structure. The page has two top-level sections (`#home-section` for the start screen and `#result-section` for the result view), and loads the SDK bundle from the `dynamsoft-mrz-scanner` folder served at `/`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner</title>
    <script src="/dynamsoft-mrz-scanner/dist/mrz-scanner.bundle.js"></script>

    <!-- Demo stylesheets — copy from samples/demo/css/ in the GitHub repo:
         https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/demo/css
         The markup below uses the class names these files expect. Without
         them the result view will render unstyled. -->
    <link rel="stylesheet" href="css/index.css" />
    <link rel="stylesheet" href="css/result.css" />

    <style>
      /* Minimal home-section styling so the entry-point buttons remain
         visible even if the demo's stylesheets above aren't linked. */
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

## Step 2: Initialize the Scanner

Inside the `<script type="module">` block, declare a module-level `mrzScanner` variable and create the scanner instance:

```js
let mrzScanner;

try {
  mrzScanner = new Dynamsoft.MRZScanner({
    license: "YOUR_LICENSE_KEY_HERE",
    engineResourcePaths: {
      dcvBundle: "/dynamsoft-capture-vision-bundle/dist/",
      dcvData: "/dynamsoft-capture-vision-data/",
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

- **`license`** — replace `YOUR_LICENSE_KEY_HERE` with your trial or full key. An invalid license causes a launch error.
- **`engineResourcePaths`** — tells the SDK where to find the DCV engine bundle and model data files. These paths are **required** when installing via npm. The values above assume the three Dynamsoft folders are staged under `public/` and served at `/` (per the [User Guide]({{ site.guides }}mrz-scanner.html#step-2-initialize-the-scanner)). Omitting `engineResourcePaths` causes a resource-initialization error at launch.
- **`returnOriginalImage` / `returnDocumentImage` / `returnPortraitImage`** — control which image artifacts are attached to the result. `returnDocumentImage` and `returnPortraitImage` default to `true`; `returnOriginalImage` defaults to `false`. All three are set explicitly here so the result view can render every image kind. Setting `returnPortraitImage: false` also disables multi-side scanning. See the [User Guide]({{ site.guides }}mrz-scanner.html#multi-side-scanning) for details.

For the full list of configuration options, see the [MRZScannerConfig API]({{ site.api }}mrz-scanner.html#mrzscannerconfig).

## Step 3: Wire the Camera Scan Entry Point

Add a `startScanning` function that disables both entry-point buttons during the scan, calls `mrzScanner.launch()` to open the live-camera UI, and hands the result off to a `displayResults` function (defined in Step 5):

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

`launch()` opens the **MRZScannerView**, a full-screen container with a live camera feed, a guide frame, format selector, and toolbar buttons. When an MRZ is recognized, the promise resolves with an [**`MRZResult`**]({{ site.api }}mrz-scanner.html#mrzresult). When the user closes the scanner without scanning, the promise still resolves but with `result.data` undefined, which the `displayResults` function handles in Step 5.

## Step 4: Wire the File-Upload Entry Point

`launch()` accepts an optional `Blob`, `File`, image URL, or HTML media element. Passing one skips the camera and processes the image directly. Wire up the upload button to a hidden `<input type="file">`, await the user's selection, and pass the resulting `File` to `launch()`:

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

## Step 5: Render the Result

The `displayResults` function takes the `MRZResult` returned by `launch()` and populates the result view. It handles three concerns:

1. **Parsed data** — fields read from `result.data`.
2. **Images** — read from `result.getDocumentImage(side)`, `result.getOriginalImage(side)`, and `result.getPortraitImage()`. Details on the return type and `null` cases are in the Key APIs reference below.
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
- **`result.getDocumentImage(side)`** — deskewed crop of the document for the given side. `Dynamsoft.EnumDocumentSide.MRZ` is the side carrying the MRZ; `EnumDocumentSide.Opposite` is the opposite side, populated only when multi-side scanning runs.
- **`result.getOriginalImage(side)`** — the full unmodified frame for the given side. Only populated when `returnOriginalImage: true`.
- **`result.getPortraitImage()`** — the portrait crop, regardless of which side it was found on.
- **`DSImageData.toCanvas()`** — converts the image into an `HTMLCanvasElement` ready to append to the DOM. `toBlob()` is also available if you'd rather upload or store the image.

## Step 6: Wire Re-Scan and Return Home

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
> If you need to **change the configuration** (different `mrzFormatType`, different `return*Image` flags, different `scannerViewConfig`) between scans, replace the instance instead of reusing it: `mrzScanner = new Dynamsoft.MRZScanner({ ...newConfig })` and then call `launch()`. Configuration is fixed at construction time. See the [User Guide]({{ site.guides }}mrz-scanner.html#lifecycle-and-disposal) for the full set of patterns.

## Putting It All Together

The full `index.html` after all six steps. Before running this, make sure you've copied [`samples/demo/css/index.css`](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/css/index.css) and [`samples/demo/css/result.css`](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/css/result.css) into a `css/` folder next to this file, since the `<link>` tags in `<head>` reference them. Without those files the markup still works, but the result view renders unstyled.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner</title>
    <script src="/dynamsoft-mrz-scanner/dist/mrz-scanner.bundle.js"></script>

    <!-- Demo stylesheets — copy from samples/demo/css/ in the GitHub repo:
         https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/demo/css
         The markup below uses the class names these files expect. Without
         them the result view will render unstyled. -->
    <link rel="stylesheet" href="css/index.css" />
    <link rel="stylesheet" href="css/result.css" />

    <style>
      /* Minimal home-section styling so the entry-point buttons remain
         visible even if the demo's stylesheets above aren't linked. */
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
            dcvBundle: "/dynamsoft-capture-vision-bundle/dist/",
            dcvData: "/dynamsoft-capture-vision-data/",
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

## Next Steps

- For the full demo source (branded landing page, QR-code handoff, info-menu dropdown, and full theming), see [`samples/demo/`](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/demo) in the GitHub repository.
- For the production npm + `public/` setup that backs this walkthrough, see the [User Guide]({{ site.guides }}mrz-scanner.html).
- To customize the scanner UI, theme, and behavior, see the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).
