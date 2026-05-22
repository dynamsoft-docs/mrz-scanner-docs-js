---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition - Quick Start
keywords: Documentation, MRZ Scanner, Dynamsoft MRZ Scanner JavaScript Edition, Quick Start, Hello World
description: Get a Dynamsoft MRZ Scanner Hello World page running from your local disk in minutes using the jsDelivr CDN.
permalink: /guides/mrz-scanner-quick-start.html
---

# Quick Start for the MRZ Scanner JavaScript Edition

This Quick Start walks you through building a single-file Hello World page that scans MRZ (Machine Readable Zone) documents directly from your local disk. The MRZ Scanner is loaded via the jsDelivr CDN — no project to scaffold, no package to install, no server to run.

> [!IMPORTANT]
> The Hello World is intentionally minimal and applies **no styling**. The captured document image, portrait crop, and parsed fields render as raw, unstyled HTML, so the result view will look rough and the images may stack or size awkwardly. For a styled, production-ready result view, follow the full [User Guide]({{ site.guides }}mrz-scanner.html).

> [!TIP]
> Targeting a production deployment with npm, framework integration, or HTTPS hosting? See the full [User Guide]({{ site.guides }}mrz-scanner.html).

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

You'll need:

- **A Chromium-based browser** (Google Chrome or Microsoft Edge). The Hello World page in this guide is opened directly from disk via a `file:///` URL, and only Chromium-based browsers grant camera access in that context.
- **A trial or full license key** (see [License](#license)).

> [!WARNING]
> **Safari** does not grant camera access on `file:///` URLs at all. **Firefox** blocks it by default. To run this Quick Start, use **Chrome** or **Edge**. Browser policies around `file:///` secure contexts may change over time; if you run into issues, fall back to serving the file from a local web server as described in the full [User Guide]({{ site.guides }}mrz-scanner.html).

## Including the Library

The Quick Start loads the MRZ Scanner SDK from a CDN, so a single `<script>` tag in your HTML's `<head>` is all that's needed. Use either [jsDelivr](https://www.jsdelivr.com/) or [unpkg](https://unpkg.com/):

```html
<!-- jsDelivr -->
<script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@4.0.0/dist/mrz-scanner.bundle.js"></script>

<!-- or, unpkg -->
<script src="https://unpkg.com/dynamsoft-mrz-scanner@4.0.0/dist/mrz-scanner.bundle.js"></script>
```

> [!TIP]
> For production, install the SDK from npm and self-host the runtime assets instead — see the full [User Guide]({{ site.guides }}mrz-scanner.html).

## Step 1: Create the Hello World File

Create a new file named `hello-world.html` anywhere on your machine (e.g. on your desktop) and paste the following into it:

```html
<!doctype html>
<html lang="en">
	<head>
		<meta charset="utf-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />
		<title>Dynamsoft MRZ Scanner - Hello World</title>
		<script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@4.0.0/dist/mrz-scanner.bundle.js"></script>
	</head>
	<body>
		<h1>Dynamsoft MRZ Scanner</h1>
		<div id="results"></div>

		<script type="module">
			const results = document.querySelector("#results");

			const mrzscanner = new Dynamsoft.MRZScanner({
				license: "YOUR_LICENSE_KEY_HERE",
			});

			const result = await mrzscanner.launch();
			if (!result?.data) {
				results.textContent = "No MRZ scanned. Please try again.";
			} else {
				const mrzSide = result.getDocumentImage(Dynamsoft.EnumDocumentSide.MRZ);
				const portraitSide = result.getDocumentImage(Dynamsoft.EnumDocumentSide.Opposite);
				const portrait = result.getPortraitImage();
				if (portraitSide?.toCanvas) results.appendChild(portraitSide.toCanvas());
				if (mrzSide?.toCanvas) results.appendChild(mrzSide.toCanvas());
				if (portrait?.toCanvas) results.appendChild(portrait.toCanvas());

				const pre = document.createElement("pre");
				pre.textContent = JSON.stringify(result.data, null, 2);
				results.appendChild(pre);
			}
		</script>
	</body>
</html>
```

Replace `YOUR_LICENSE_KEY_HERE` with the license key from [License](#license).

> [!NOTE]
> When the SDK is loaded from a CDN like jsDelivr or unpkg, it auto-resolves its UI templates, WebAssembly engine, and model data from that same CDN. No `engineResourcePaths` configuration is required for the Quick Start. For a production setup with the SDK installed via npm and resources self-hosted from your own origin, `engineResourcePaths` becomes required. See the [User Guide]({{ site.guides }}mrz-scanner.html#step-2-initialize-the-scanner) for that setup.

## Step 2: Open the File in Your Browser

Double-click `hello-world.html` to open it in your default browser, or right-click → **Open With** → **Google Chrome** / **Microsoft Edge**. The address bar should show a `file:///` URL.

Grant camera permission when prompted. The MRZ Scanner UI takes over the page, and once a passport, ID, or visa is recognized (from the live camera feed or an uploaded image), the cropped document image, portrait, and parsed fields are rendered below the heading. With no CSS applied, expect a bare, awkwardly-laid-out result view; styling is covered in the full [User Guide]({{ site.guides }}mrz-scanner.html).

## What's in the Hello World

Once `mrz-scanner.bundle.js` is loaded, it exposes a global `Dynamsoft` namespace. The page does three things:

1. **Construct an `MRZScanner`** with your license key.
2. **Call `launch()`**, which opens the full-screen scanner UI and resolves with an `MRZResult` once an MRZ is recognized (or with an empty result if the user cancels).
3. **Render the result** — the cropped document image (`getDocumentImage(EnumDocumentSide.MRZ)`), the portrait crop (`getPortraitImage()`), and the parsed fields dumped as raw JSON via `<pre>`.

For a deeper walkthrough of the API and UI, see the full [User Guide]({{ site.guides }}mrz-scanner.html).

## Next Steps

- **Production integration** — the [User Guide]({{ site.guides }}mrz-scanner.html) covers npm-based installation, deployment requirements, and serving over HTTPS with Vite.
- **Customizing the scanner UI and behavior** — see the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).
- **Scanning from static images and PDFs** — see [Setting up the MRZ Scanner for Static Images and PDFs]({{ site.guides }}mrz-scanner-static-image.html).
