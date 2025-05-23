---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: false
title: Dynamsoft MRZ Scanner JavaScript Edition - API Reference
keywords: Documentation, MRZ Scanner JavaScript Edition, API, APIs, Enumeration, Enums, Enum
breadcrumbText: API References
description: Dynamsoft MRZ Scanner JavaScript Edition - API Reference
---

# MRZ Scanner API Enumerations

The MRZ Scanner comes with a few enumerations for some of the configuration properties.

## EnumMRZDocumentType

### Syntax

```ts
enum EnumMRZDocumentType {
  Passport = "passport",
  TD1 = "td1",
  TD2 = "td2",
}
```

## EnumResultStatus

### Syntax

```ts
enum EnumResultStatus {
  RS_SUCCESS = 0,
  RS_CANCELLED = 1,
  RS_FAILED = 2,
}
```

## EnumMRZData

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
  Nationality = "nationality",
  DateOfBirth = "dateOfBirth",
  DateOfExpiry = "dateOfExpiry",
}
```
