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

1. [Can the MRZ Scanner process static images, or does it only work with live cameras?](#can-the-mrz-scanner-process-static-images-or-does-it-only-work-with-live-cameras)
2. [Does MRZ Scanner perform data validation?](#does-the-mrz-scanner-perform-data-validation)
3. [How do I use the scanned result?](#how-do-i-use-the-scanned-result)

## Can the MRZ Scanner process static images, or does it only work with live cameras?

Yes, the MRZ Scanner can read from static images and image files, along with the typical use case of scanning from a live camera feed.

## Does the MRZ Scanner perform data validation?

**No**. MRZ Scanner JavaScript Edition performs all image capture, image enhancement, and MRZ parsing on-device, to give you full control of your data. The MRZ Scanner processes the data on the client device without passing the data to external servers. Your application must implement data validation on the decoded MRZ data after reading with the MRZ Scanner if your use case requires data validation.


## How do I use the scanned result?

The simplest implementation uses the default UI and does not take action after scanning. You can customize the behavior by handling callbacks provided by the API. The [`launch()`]({{ site.api }}mrz-scanner.html#launch) method returns the scanned result as the type [`MRZResult`]({{ site.api }}mrz-scanner.html#mrzresult), which includes the image of the document, as well as the parsed MRZ data separated into fields.
