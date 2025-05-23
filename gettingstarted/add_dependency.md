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

To build the solution, we need to include six packages

1. `dynamsoft-label-recognizer`: Required. It defines the classes, interfaces, and enumerations specifically tailored for the Label Recognizer module.
2. `dynamsoft-core`: Required. It encompasses common classes, interfaces, and enumerations that are shared across all SDKs in the Capture Vision architecture.
3. `dynamsoft-license`: Required. It is responsible for the operations needed to initialize and operate the license.
4. `dynamsoft-code-parser`: Required. It defines interfaces and enumerations specifically tailored to the Code Parser module.
5. `dynamsoft-capture-vision-router`: Required. It defines the class `CaptureVisionRouter`, which governs the entire image processing workflow.
6. `dynamsoft-camera-enhancer`: Required. It is used to define camera control features and capbilities as well as image acquisition features.

## Use a CDN

The simplest way to include the SDK is to use either the [jsDelivr](https://jsdelivr.com/) or [UNPKG](https://unpkg.com/) CDN.

- jsDelivr

  ```html
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@2.0.0/dist/ddv.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-core@3.2.10/dist/core.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-license@3.2.10/dist/license.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-document-normalizer@2.2.10/dist/ddn.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-router@2.2.10/dist/cvr.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/dynamsoft-camera-enhancer@4.0.2/dist/dce.js"></script>
  ```

- UNPKG

  ```html
  <script src="https://unpkg.com/dynamsoft-document-viewer@2.0.0/dist/ddv.js"></script>
  <script src="https://unpkg.com/dynamsoft-core@3.2.10/dist/core.js"></script>
  <script src="https://unpkg.com/dynamsoft-license@3.2.10/dist/license.js"></script>
  <script src="https://unpkg.com/dynamsoft-document-normalizer@2.2.10/dist/ddn.js"></script>
  <script src="https://unpkg.com/dynamsoft-capture-vision-router@2.2.10/dist/cvr.js"></script>
  <script src="https://unpkg.com/dynamsoft-camera-enhancer@4.0.2/dist/dce.js"></script>
  ```

## Host yourself

Besides using the CDN, you can also download the SDKs and host related files on your own website/server before including it in your application. When using a CDN, resources related to `dynamsoft-image-processing` and `dynamsoft-capture-vision-std` are automatically loaded over the network; When using them locally, these two packages need to be configured manually.

**Step 1** - Download the SDKs:
{% comment %}

- From the website

  [Download the JavaScript ZIP package](https://www.dynamsoft.com/mobile-web-capture/downloads/)

- yarn

  ```cmd
  yarn add dynamsoft-document-viewer@2.0.0
  yarn add dynamsoft-core@3.2.10
  yarn add dynamsoft-license@3.2.10
  yarn add dynamsoft-document-normalizer@2.2.10
  yarn add dynamsoft-capture-vision-router@2.2.10
  yarn add dynamsoft-capture-vision-std@1.2.0
  yarn add dynamsoft-image-processing@2.2.10
  yarn add dynamsoft-camera-enhancer@4.0.2
  ```

  {% endcomment %}

- npm

  ```cmd
  npm install dynamsoft-document-viewer@2.0.0
  npm install dynamsoft-core@3.2.10
  npm install dynamsoft-license@3.2.10
  npm install dynamsoft-document-normalizer@2.2.10
  npm install dynamsoft-capture-vision-router@2.2.10
  npm install dynamsoft-capture-vision-std@1.2.0
  npm install dynamsoft-image-processing@2.2.10
  npm install dynamsoft-camera-enhancer@4.0.2
  ```

**Step 2** - Include the SDKs

Depending on where you put them, you can typically include them like this:
{% comment %}

```html
<script src="./distributables/dynamsoft-document-viewer@2.0.0/dist/ddv.js"></script>
<script src="./distributables/dynamsoft-core@3.2.10/dist/core.js"></script>
<script src="./distributables/dynamsoft-license@3.2.10/dist/license.js"></script>
<script src="./distributables/dynamsoft-document-normalizer@2.2.10/dist/ddn.js"></script>
<script src="./distributables/dynamsoft-capture-vision-router@2.2.10/dist/cvr.js"></script>
<script src="./distributables/dynamsoft-camera-enhancer@4.0.2/dist/dce.js"></script>
```

or
{% endcomment %}

```html
<script src="./node_modules/dynamsoft-document-viewer/dist/ddv.js"></script>
<script src="./node_modules/dynamsoft-core/dist/core.js"></script>
<script src="./node_modules/dynamsoft-license/dist/license.js"></script>
<script src="./node_modules/dynamsoft-document-normalizer/dist/ddn.js"></script>
<script src="./node_modules/dynamsoft-capture-vision-router/dist/cvr.js"></script>
<script src="./node_modules/dynamsoft-camera-enhancer/dist/dce.js"></script>
```

**Step 3** Specify the location of the engine files(optinal)

If you would like to use the SDKs completely offline, please refer to [Use your own hosted engine files](#use-your-own-hosted-engine-files).

## Specify the location of the engine files

This is usually only required with frameworks like Angular or React, etc. where the referenced JavaScript files such as cvr.js, ddn.js are compiled into another file, or hosting the engine files and using the SDKs completely offline. The purpose is to tell the SDK where to find the engine files (_.worker.js, _.wasm.js and \*.wasm, etc.).

### Use the jsDelivr CDN with frameworks like Angular or React, etc.

```typescript
Dynamsoft.DDV.Core.engineResourcePath =
  "https://cdn.jsdelivr.net/npm/dynamsoft-document-viewer@2.0.0/dist/engine";
Dynamsoft.Core.CoreModule.engineResourcePaths.core =
  "https://cdn.jsdelivr.net/npm/dynamsoft-core@3.2.10/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.license =
  "https://cdn.jsdelivr.net/npm/dynamsoft-license@3.2.10/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.ddn =
  "https://cdn.jsdelivr.net/npm/dynamsoft-document-normalizer@2.2.10/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.cvr =
  "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-router@2.2.10/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.std =
  "https://cdn.jsdelivr.net/npm/dynamsoft-capture-vision-std@1.2.0/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dip =
  "https://cdn.jsdelivr.net/npm/dynamsoft-image-processing@2.2.10/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dce =
  "https://cdn.jsdelivr.net/npm/dynamsoft-camera-enhancer@4.0.2/dist/";
```

### Use your own hosted engine files

```typescript
//Feel free to change it to your own location of these files
Dynamsoft.DDV.Core.engineResourcePath =
  "./node_modules/dynamsoft-document-viewer/dist/engine";
Dynamsoft.Core.CoreModule.engineResourcePaths.core =
  "./node_modules/dynamsoft-core/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.license =
  "./node_modules/dynamsoft-license/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.ddn =
  "./node_modules/dynamsoft-document-normalizer/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.cvr =
  "./node_modules/dynamsoft-capture-vision-router/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.std =
  "./node_modules/dynamsoft-capture-vision-std/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dip =
  "./node_modules/dynamsoft-image-processing/dist/";
Dynamsoft.Core.CoreModule.engineResourcePaths.dce =
  "./node_modules/dynamsoft-camera-enhancer/dist/";
```
