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

Using the Hello World implementation as a reference, the Demo code offers a bit more colour and styling to the MRZ Scanner implementation.

## Code

The full code of the demo can be found [**here**](https://github.com/Dynamsoft/mrz-scanner-javascript/blob/main/samples/demo/index.html).

Please note that this code uses the library via local sources, meaning that the library is built from the source files that are in the same repo.

In order to run the app, all you need to do is this:

1. Download/Clone the repository via Github

2. Execute the `npm run build` command in the terminal (or Powershell)

3. Execute the `npm run serve` command

After that, you should get a couple of URLs, one for the Hello World sample and one for the Demo sample.

## Live Demo

For the best demonstration of the full capabilities of the MRZ Scanner JavaScript Edition, please visit the [official Dynamsoft MRZ Scanner JavaScript demo](https://demo.dynamsoft.com/mrz-scanner/). Please note that the official demo includes the usage of some "virtual" cameras, so the code shared on this page is slightly different than the official demo. However, in terms of the operation of the MRZ Scanner itself, the code is nearly identical.

## Conclusion

If you do not wish to build the library from the source files, you can change the script reference at the `<head>` of the code to the CDN link. To learn more about the ways you can include the library, please visit [User Guide - Quick Start]({{ site.guides }}mrz-scanner.html#quick-start---hello-world).

To learn how you can further customize the MRZ Scanner in your code, please visit [Customizing the MRZ Scanner]({{ site.guides }}mrz-scanner-customization.html).
