---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - License
keywords: Documentation, MRZ Scanner JavaScript Edition, License
breadcrumbText: License
description: MRZ Scanner JavaScript Edition Documentation License
permalink: /gettingstarted/license.html
---

# License

## Get a trial key

- A free public trial license is available for every new device for first use of the MRZ Scanner JavaScript Edition. The public trial license is the default key used in samples. You can also find the public trial license on the following parts of this page.
- If your free key is expired, please visit <a href="https://www.dynamsoft.com/customer/license/trialLicense?product=mwc&source=docs" target="_blank">Private Trial License Page</a> to get a 30-day trial extension.

## Get a full license

- [Contact us](https://www.dynamsoft.com/company/contact/) to purchase a full license

## Initialize the license

The following code snippet is using the public trial license to initialize the license. You can replace the public trial license with your own license key.

```ts
const mrzScanner = new Dynamsoft.MRZScanner({
    license: "YOUR_LICENSE_KEY_HERE",
});
```
