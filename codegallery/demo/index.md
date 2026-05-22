---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - Demo
keywords: Documentation, MRZ Scanner JavaScript Edition, Demo
breadcrumbText: Demo
description: MRZ Scanner JavaScript Edition Documentation Demo
permalink: /codegallery/demo/index.html
---

# MRZ Scanner JavaScript Edition - Demo

The Dynamsoft MRZ Scanner Demo is a fully styled, mobile-and-desktop-responsive reference implementation. It pairs the SDK with a branded landing page, a desktop-to-mobile QR-code handoff, and a polished launch flow — all built on the same `MRZScanner` / `launch()` API used in the [Quick Start]({{ site.guides }}mrz-scanner-quick-start.html) and the [User Guide]({{ site.guides }}mrz-scanner.html).

## Source

The full source lives at [`samples/demo/`](https://github.com/Dynamsoft/mrz-scanner-javascript/tree/main/samples/demo) in the [`Dynamsoft/mrz-scanner-javascript`](https://github.com/Dynamsoft/mrz-scanner-javascript) repository.

## Running the Demo Locally

1. **Clone the repository** (or download a ZIP and extract it):

    ```bash
    git clone https://github.com/Dynamsoft/mrz-scanner-javascript.git
    ```

2. **Move into the demo folder, install dependencies, build, and start the dev server:**

    ```bash
    cd mrz-scanner-javascript/samples/demo
    npm install
    npm run build
    npm run dev
    ```

The dev server prints two URLs — a `https://localhost:<port>` URL for the host machine, and a `https://<lan-ip>:<port>` URL for other devices on the same network. Open either, accept the self-signed certificate warning on first visit, and grant camera permission to use the demo.

## Live Demo

Try the [official Dynamsoft MRZ Scanner demo](https://demo.dynamsoft.com/mrz-scanner/) if you want to see the user flow before cloning.

## Next Steps

- For a step-by-step walkthrough of the SDK API, see the [Quick Start]({{ site.guides }}mrz-scanner-quick-start.html).
- For a full production integration via npm, see the [User Guide]({{ site.guides }}mrz-scanner.html).
- To customize the scanner UI, theme, and behavior, see the [Customization Guide]({{ site.guides }}mrz-scanner-customization.html).
