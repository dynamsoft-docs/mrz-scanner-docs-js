---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - System Requirements
keywords: Documentation, MRZ Scanner, Web, System Requirements
breadcrumbText: System Requirements
description: MRZ Scanner JavaScript Edition Documentation System Requirements
permalink: /gettingstarted/sys_requirement.html
---

# System Requirements

The MRZ Scanner JavaScript Edition solution is built upon the JavaScript edition of two foundational SDK products, the <a href="https://www.dynamsoft.com/label-recognition/overview/" target="_blank">Dynamsoft Label Recognizer</a> (DLR) and the <a href="https://www.dynamsoft.com/label-recognition/overview/" target="_blank">Dynamsoft Code Parser</a> (DCP). In order to make all of the features of this solution work, the following requirememts are needed:

1. **Secure context (HTTPS deployment)**

    When deploying your application / website for production, make sure to serve it via a secure HTTPS connection. This is required for two reasons

  - Access to the camera video stream is only granted in a security context. Most browsers impose this restriction.
    > Some browsers like Chrome may grant the access for `http://127.0.0.1` and `http://localhost` or even for pages opened directly from the local disk (`file:///...`). This can be helpful for temporary development and testing.

  - Dynamsoft licenses require a secure context to work, unless a special offline license is used.

2. **`WebAssembly`, `Blob`, `URL`/`createObjectURL`, `Web Workers`**

  The above four browser features are required for the SDKs to work.

## Supported Browsers

The following table is a list of supported browsers based on the above requirements:

  | Browser Name |             Version              |
  | :----------: | :------------------------------: |
  |    Chrome    |             v78+                 |
  |   Firefox    |             v79+ (v55+ on iOS/Android)                 |
  |    Safari    |             v15+                 |
  |     Edge     |             v92+                 |

Apart from the browsers, the operating systems may impose some limitations of their own that could restrict the use of the SDKs.

>Note:
>
> iOS 14.3+ is required for camera video streaming in Chrome and Firefox or apps using Webviews.

## References

- [Dynamsoft Label Recognizer - System Requirements](https://www.dynamsoft.com/capture-vision/docs/web/programming/javascript/user-guide/mrz-scanner.html#system-requirements)
- [Dynamsoft Code Parser - System Requirements](https://www.dynamsoft.com/code-parser/docs/web/programming/javascript/user-guide/#system-requirements)
