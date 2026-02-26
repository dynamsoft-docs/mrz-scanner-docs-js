---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - Adding the dependency
keywords: Documentation, MRZ Scanner JavaScript Edition, Adding the dependency
breadcrumbText: Adding the dependency
description: MRZ Scanner JavaScript Edition Documentation Adding the dependency
permalink: /gettingstarted/add_dependency.html
---

# Adding the Dependency

To build a MRZ Scanner web app, include the `dynamsoft-mrz-scanner` SDK.

## Use a CDN

The simplest way to include the SDK is to use either the [jsDelivr](https://jsdelivr.com/) or [UNPKG](https://unpkg.com/) CDN.

- jsDelivr

  ```html
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@3.1.0/dist/mrz-scanner.bundle.js"></script>
  ```

- UNPKG

  ```html
  <script src="https://unpkg.com/dynamsoft-mrz-scanner@3.1.0/dist/mrz-scanner.bundle.js"></script>
  ```

## Use npm or yarn

When using frameworks such as React, Vue, or Angular, add the package as a project dependency:

```sh
npm i dynamsoft-mrz-scanner@3.1.0 -E
# or
yarn add dynamsoft-mrz-scanner@3.1.0 -E
```

## Specify the location of engine files (framework usage)

When the package is bundled by a build tool, set `engineResourcePaths` so the scanner can find engine files:

```typescript
Dynamsoft.Core.CoreModule.engineResourcePaths.core =
  "https://cdn.jsdelivr.net/npm/dynamsoft-core/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.license =
  "https://cdn.jsdelivr.net/npm/dynamsoft-license/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.cvr =
  "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-router/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dlr =
  "https://cdn.jsdelivr.net/npm/dynamsoft-label-recognizer/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dcp =
  "https://cdn.jsdelivr.net/npm/dynamsoft-code-parser/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.std =
  "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-std/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dip =
  "https://cdn.jsdelivr.net/npm/dynamsoft-image-processing/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dce =
  "https://cdn.jsdelivr.net/npm/dynamsoft-camera-enhancer/dist/";
```

For complete setup and usage examples, see the [User Guide]({{ site.guides }}mrz-scanner.html).
