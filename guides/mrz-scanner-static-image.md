---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Setting up a MRZ Scanner for Static Images and PDFs
keywords: Documentation, MRZ Scanner, Dynamsoft MRZ Scanner JavaScript Edition, Static Image, PDF
description: Dynamsoft MRZ Scanner User Guide
permalink: /guides/mrz-scanner-static-image.html
---

# Setting up the MRZ Scanner for Static Images and PDFs

In the main [MRZ Scanner JS User Guide]({{ site.guides }}mrz-scanner.html), we explored how to use the default UI of the MRZ Scanner to read MRZs via interactive video stream while at the same time including the ability to read from photos in your photo library via the Load Image button that is part of the [`MRZScannerView`]({{ site.guides }}mrz-scanner.html#mrzscannerview).

In **v2.1** (and beyond) of the MRZ Scanner, the library has introduced the ability to read MRZs directly from static images without the need to use the file/photo picker that is a default part of the mobile device or the computer. In this guide, we will explore how to use the MRZ Scanner API in order to read MRZs from multiple image formats, as well as PDFs.

> [!NOTE]
> In order to follow with this guide, please refer to the [**use-file-input sample**](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/scenarios/use-file-input.html) that is hosted on the main Github repo of the MRZ Scanner source files.

## Prerequisites

To get started with the development process, a valid license key is needed. To obtain a license key, please refer to the [Licensing]({{ site.guides }}mrz-scanner.html#license) section of the main User Guide. The next section will break down the code and take you through the sample step-by-step.

## Breaking Down the Code

### Step 1: Including the Library and Defining UI Elements

The first step in making the sample is to define the script references and the HTML elements of the page. In terms of the scripts, similar to the Hello World sample, the MRZ Scanner script needs to be referenced, either locally or via a CDN link. Since this sample will also support PDFs, we must use the [**PDF JS**](https://github.com/mozilla/pdf.js) library. Here is the start:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner - Use File Input</title>
    <!-- <script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@3.0.1/dist/mrz-scanner.bundle.js"></script> -->
    <!-- To use locally: -->
    <script src="../../dist/mrz-scanner.bundle.js"></script>

    <!-- PDF.js library  -->
    <script src="https://cdn.jsdelivr.net/npm/pdfjs-dist@3.11.174/build/pdf.min.js"></script>

    <style>
      html,
      body {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      #results canvas {
        width: 100%;
        height: 100%;
      }
    </style>
  </head>

  <body>
    <button type="button" id="start-scan">Start Scan Button</button>
    <input type="file" id="initialFile" accept="image/png,image/jpeg,application/pdf" />
    <div id="results"></div>
  </body>
</html>
```

### Step 2: Setting up PDF.js

Now that the HTML and scripts have been set, it's time to set up the PDF.js library in order to support loading PDFs into your web app. Here is the code to do that:

```html
<script>
    // Setup PDF.js
    const pdfjsLib = window["pdfjs-dist/build/pdf"];
    pdfjsLib.GlobalWorkerOptions.workerSrc =
    "https://cdn.jsdelivr.net/npm/pdfjs-dist@3.11.174/build/pdf.worker.min.js";

    const resultContainer = document.querySelector("#results");
</script>
```

### Step 3: Initializing the Dynamsoft MRZ Scanner

Now that we are working on the main script of the operation of the web application, one of the first things that we need to do is to initialize the MRZ Scanner which will be responsible for the MRZ reading functionality of the web app.

```js
// Initialize the Dynamsoft MRZ Scanner
const mrzscanner = new Dynamsoft.MRZScanner({
    license: "YOUR_LICENSE_KEY_HERE",
    scannerViewConfig: {
        uploadAcceptedTypes: "image/*,application/pdf",
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
    },
});
```

It is important to pay attention to the new properties that have been introduced to the [**`MRZScannerViewConfig`**]({{ site.api }}mrz-scanner.html#mrzscannerviewconfig) interface. The first new property is `uploadAcceptedTypes` which sets the file format(s) that the MRZ Scanner will accept. In this sample, we set to accept any image type as well as PDF. The second property is the `uploadFileConverter` - which is needed to support PDF files as those need to be converted to images so that the MRZ Scanner can accept them as input.

### Step 4: Implementing the convertPDFToImage function

In the previous step, we used a `convertPDFToImage` function in the `uploadFileConverter` property, but that function has not been implemented yet. Let's implement that now:

```js
// PDF to image conversion function
async function convertPDFToImage(file) {
    try {
        // Load the PDF file
        const arrayBuffer = await file.arrayBuffer();
        const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;

        // Get the first page only
        if (pdf.numPages === 0) {
            throw new Error("The PDF has no pages");
        }

        const page = await pdf.getPage(1); // Page numbers are 1-based in pdf.js

        // Set a reasonable scale for good rendering quality
        const scale = 1.5;
        const viewport = page.getViewport({ scale });

        // Create a canvas for rendering
        const canvas = document.createElement("canvas");
        const context = canvas.getContext("2d");
        canvas.width = viewport.width;
        canvas.height = viewport.height;

        // Render the PDF page to the canvas
        const renderContext = {
            canvasContext: context,
            viewport: viewport,
        };

        await page.render(renderContext).promise;

        // Convert canvas to blob
        return new Promise((resolve, reject) => {
        canvas.toBlob((blob) => {
            if (blob) {
                resolve(blob);
            } else {
                reject(new Error("Failed to create image from PDF"));
            }
        }, "image/png");
        });
    } catch (error) {
        console.error("PDF processing error:", error);
        alert("PDF Processing error. Please try again");
        throw new Error("PDF Processing Error.");
    }
}
```

> [!NOTE]
> It is important to note that currently, this sample only accepts **single-page PDF** files. The above `convertPDFToImage` function can process only a single-page PDF file.

In this function, any **single-page** PDF file that is accepted as input is converted to a Blob that has a `PNG` type. Now that the PDF has been converted to a an image type, it's ready to get fed to the MRZ Scanner to be read.

### Step 5: Launching the MRZ Scanner

Now that all of the core functions needed for loading the image or PDF have been implemented, it's time to connect all of these parts to the [`launch`]({{ site.api }}mrz-scanner.html#launch) method of the MRZ Scanner. One of the main changes in v2.1 of the MRZ Scanner is that the `launch` method can now be run with a file input. 

In the code below, there are two trigger functions, one for launching the MRZ Scanner with its normal camera UI (MRZScannerView) The following trigger functions demonstrate how to read the MRZ from a file when said file is uploaded, then we automatically run the `launch` command in order to read the MRZ from the file without needing to click "Start Scan".

```js
document.getElementById("start-scan").onclick = async function () {
    const result = await mrzscanner.launch();
    displayResult(result);
};

document.getElementById("initialFile").onchange = async function () {
    const files = Array.from(this.files || []);
    if (files.length) {
        try {
            let fileToProcess = files[0];

            // Handle PDF conversion if needed
            if (fileToProcess.type === "application/pdf") {
                fileToProcess = await convertPDFToImage(fileToProcess);
            }

            const result = await mrzscanner.launch(fileToProcess);
            displayResult(result);
        } catch (error) {
            console.error("Error processing file:", error);
            resultContainer.innerHTML = `<p>Error: ${error.message}</p>`;
        }
    }
};
```

> [!NOTE]
> Please note the difference in how the launch command is run when using the default UI (which includes the camera) and how the launch command is run when trying to read from a static image/file. When using the default UI, it is run as  `mrzScanner.launch({})`. When reading from a static image, it is run as `mrzScanner.launch()` or `mrzScanner.launch(file)`.

### Step 6: Displaying the Result

When the MRZ is read, the last step that we need to do is to display the result to the user. Here is the `displayResult` function

```js
function displayResult(result) {
    console.log(result);

    if (result?.data) {
        resultContainer.innerHTML = ""; // Clear placeholder content

        if (result.originalImageResult?.toCanvas) {
        const canvas = result.originalImageResult?.toCanvas();

        canvas.style.objectFit = "contain";
        canvas.style.width = "100%";
        canvas.style.height = "100%";
        resultContainer.appendChild(canvas);
        }

        let resultHTML = ``;
        Object.entries(result.data).forEach(([key, value]) => {
            const label = Dynamsoft.MRZDataLabel[key];

            const container = document.createElement("div");
            container.className = "mrz-result-container";
            const labelContainer = document.createElement("div");
            const valueContainer = document.createElement("div");

            labelContainer.textContent = label;
            valueContainer.textContent = `${JSON.stringify(value)}`;

            container.appendChild(labelContainer);
            container.appendChild(valueContainer);
            resultContainer.appendChild(container);
        });
    } else {
        resultContainer.innerHTML = "<p>No MRZ scanned. Please try again.</p>";
    }
}
```

## Conclusion

Now that the code is all done, all you need to do is run the app either via npm or you can host the source files yourself and serve your web application via a HTTPS environment.

With this app, you can read MRZs from static images and files without needing to use the MRZScannerView UI, therefore covering cases where a camera is not needed at all or cases where the images are obtained via a different source e.g. via a scanner and Dynamic Web TWAIN.

If you have any questions about this use case, please contact the [Dynamsoft Support Team](https://www.dynamsoft.com/company/contact/).