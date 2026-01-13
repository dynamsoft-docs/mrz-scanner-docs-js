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

# Using the MRZ Scanner with Static Images and PDFs

The main [MRZ Scanner User Guide]({{ site.guides }}mrz-scanner.html) demonstrates scanning MRZs from a live camera feed, including the Load Image button in the [`MRZScannerView`]({{ site.guides }}mrz-scanner.html#mrzscannerview) for selecting photos from your device.

Starting with **v2.1**, the MRZ Scanner can read MRZs directly from static images and PDFs without requiring the default file picker. This guide shows you how to implement this functionality programmatically, supporting multiple image formats and PDF documents.

> [!NOTE]
> To follow along with this guide, refer to the [use-file-input sample](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/scenarios/use-file-input.html) in the MRZ Scanner GitHub repository.

## Prerequisites

You'll need a valid license key to get started. Refer to the [Licensing]({{ site.guides }}mrz-scanner.html#license) section of the main User Guide for instructions on obtaining one.

## Understanding the Implementation

### Step 1: Including the Library and Defining UI Elements

The first step is to include the required libraries and define the HTML elements. You'll need:

1. The MRZ Scanner library (via CDN or local reference)
2. The [PDF.js library](https://github.com/mozilla/pdf.js) for PDF support

Here's the basic HTML structure:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner - Use File Input</title>
    <!-- <script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@3.1.0/dist/mrz-scanner.bundle.js"></script> -->
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

### Step 2: Configuring PDF.js

Configure the PDF.js library to enable PDF loading in your application:

```html
<script>
    // Setup PDF.js
    const pdfjsLib = window["pdfjs-dist/build/pdf"];
    pdfjsLib.GlobalWorkerOptions.workerSrc =
    "https://cdn.jsdelivr.net/npm/pdfjs-dist@3.11.174/build/pdf.worker.min.js";

    const resultContainer = document.querySelector("#results");
</script>
```

### Step 3: Initializing the MRZ Scanner

Initialize the MRZ Scanner with custom configuration to handle static images and PDFs:

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

Note the new properties in [**`MRZScannerViewConfig`**]({{ site.api }}mrz-scanner.html#mrzscannerviewconfig):

- **`uploadAcceptedTypes`**: Specifies accepted file formats (images and PDFs in this example)
- **`uploadFileConverter`**: Converts PDFs to images before processing, as the scanner requires image input

### Step 4: Implementing the PDF Conversion Function

Implement the `convertPDFToImage` function referenced in the previous step:

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
> This sample supports **single-page PDFs only**. Multi-page PDFs require additional logic to process each page.

This function converts a **single-page** PDF file to a PNG Blob, making it compatible with the MRZ Scanner.

### Step 5: Launching the MRZ Scanner

With the PDF conversion function in place, connect everything to the [`launch`]({{ site.api }}mrz-scanner.html#launch) method. Starting in v2.1, the `launch` method accepts a file input parameter.

The code below shows two ways to trigger the scanner:

1. **Camera UI**: Click the "Start Scan" button to launch the standard camera interface
2. **File Upload**: Select a file to automatically process it without showing the camera UI

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
> The `launch()` method behavior differs based on the parameter:
> - **No parameter or empty object** (`launch()` or `launch({})`): Opens the camera UI
> - **File parameter** (`launch(file)`): Processes the file directly without UI

### Step 6: Displaying the Result

Finally, display the scanned MRZ results to the user:

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

You now have a complete implementation for scanning MRZs from static images and PDFs. To run the application:

1. Serve the files via HTTPS (required for the license to work)
2. Use a local development server like the Five Server extension for VS Code, or
3. Deploy to your web server

This approach is ideal when:

- Camera access is not needed
- Images come from external sources (scanners, document management systems, etc.)
- You want programmatic control over the scanning process

For questions or support, contact the [Dynamsoft Support Team](https://www.dynamsoft.com/company/contact/).