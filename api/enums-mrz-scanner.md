---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition - API Reference
keywords: Documentation, MRZ Scanner JavaScript Edition, API, APIs, Enumeration, Enums, Enum
breadcrumbText: API References
description: "Explore MRZ Scanner values in Dynamsoft MRZ Scanner Web API and learn how they define status, configuration, and processing behavior for modern web today."
---

# MRZ Scanner API Enumerations

The MRZ Scanner exposes a small number of enumerations used by its configuration and result interfaces.

## EnumMRZDocumentType

The set of MRTD formats the scanner can be restricted to via [`MRZScannerConfig.mrzFormatType`]({{ site.api }}mrz-scanner.html#mrzscannerconfig).

### Syntax

```ts
enum EnumMRZDocumentType {
  Passport = "td3_passport",
  TD1 = "td1_id",
  TD2 = "td2_id",
  MRVA = "mrva_visa",
  MRVB = "mrvb_visa",
}
```

### Values

| Member | Value | Document type |
| --- | --- | --- |
| `Passport` | `"td3_passport"` | TD3-format passports. |
| `TD1` | `"td1_id"` | TD1-format ID cards. |
| `TD2` | `"td2_id"` | TD2-format ID cards (covers both standard TD2 and French national ID). |
| `MRVA` | `"mrva_visa"` | Type-A machine-readable visas (TD3-sized). |
| `MRVB` | `"mrvb_visa"` | Type-B machine-readable visas (TD2-sized). |

> [!TIP]
> The string values are accepted directly when configuring `mrzFormatType`, which avoids importing the enum at every call site. For example: `mrzFormatType: ["td3_passport", "td1_id"]`.

## EnumDocumentSide

Identifies the side of the document being referred to when retrieving an image from the [`MRZResult`]({{ site.api }}mrz-scanner.html#mrzresult).

### Syntax

```ts
enum EnumDocumentSide {
  MRZ = "mrz",
  Opposite = "opposite",
}
```

### Values

| Member | Value | Description |
| --- | --- | --- |
| `MRZ` | `"mrz"` | The side of the document that contains the MRZ — always captured first. |
| `Opposite` | `"opposite"` | The opposite side, populated only when multi-side scanning runs (e.g. on TD1/TD2 ID cards where the portrait is on the side opposite the MRZ). |

> [!NOTE]
> `Opposite` is `null` on every result unless multi-side scanning was triggered. See the [User Guide's Multi-Side Scanning section]({{ site.guides }}mrz-scanner.html#multi-side-scanning) for the mechanics that determine when each side is populated.

## EnumResultStatus

The set of possible status codes carried by [`MRZResult.status`]({{ site.api }}mrz-scanner.html#mrzresult).

### Syntax

```ts
enum EnumResultStatus {
  RS_SUCCESS = 0,
  RS_CANCELLED = 1,
  RS_FAILED = 2,
}
```

### Values

| Member | Value | Meaning |
| --- | --- | --- |
| `RS_SUCCESS` | `0` | The scan completed successfully and the result carries parsed data and any opted-in images. |
| `RS_CANCELLED` | `1` | The user closed the scanner without completing a scan. The result has no `data`. |
| `RS_FAILED` | `2` | An error occurred during scanning. The result has no `data`. |

## EnumMRZData

The set of field keys present on the parsed [`MRZData`]({{ site.api }}mrz-scanner.html#mrzdata) object.

### Syntax

```ts
enum EnumMRZData {
  InvalidFields = "invalidFields",
  DocumentType = "documentType",
  DocumentNumber = "documentNumber",
  MRZText = "mrzText",
  FirstName = "firstName",
  LastName = "lastName",
  Age = "age",
  Sex = "sex",
  IssuingState = "issuingState",
  IssuingStateRaw = "issuingStateRaw",
  Nationality = "nationality",
  NationalityRaw = "nationalityRaw",
  DateOfBirth = "dateOfBirth",
  DateOfExpiry = "dateOfExpiry",
  OptionalData1 = "optionalData1",
  OptionalData2 = "optionalData2",
}
```

> [!TIP]
> Use the [`MRZDataLabel`]({{ site.api }}mrz-scanner.html#mrzdatalabel) helper to map any `EnumMRZData` key to a human-readable label when rendering a result view.
