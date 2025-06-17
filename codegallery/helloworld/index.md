---
layout: default-layout
needAutoGenerateSidebar: true
needGenerateH3Content: true
noTitleIndex: true
title: MRZ Scanner JavaScript Edition - Hello World
keywords: Documentation, MRZ Scanner JavaScript Edition, Hello World
breadcrumbText: Hello World
description: MRZ Scanner JavaScript Edition Documentation Hello World
permalink: /codegallery/helloworld/index.html
---

# MRZ Scanner JavaScript Edition - Hello World

Hello World represents the most basic implementation of the MRZ Scanner. One of the many advantages of the new design of the MRZ Scanner is that it takes very few lines of code to get the scanner up and running - and that is on full display in the Hello World code!

## Code

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Dynamsoft MRZ Scanner - Hello World</title>
    <script src="https://cdn.jsdelivr.net/npm/dynamsoft-mrz-scanner@3.0.0/dist/mrz-scanner.bundle.js"></script>
  </head>

  <body>
    <h1 style="font-size: large">Dynamsoft MRZ Scanner</h1>

    <script>
      // Initialize the Dynamsoft MRZ Scanner
      const mrzScanner = new Dynamsoft.MRZScanner({
        license: "YOUR_LICENSE_KEY_HERE",
      });
      (async () => {
        // Launch the scanner and wait for the result
        const result = await mrzScanner.launch();
      })();
    </script>
  </body>
</html>
```

## Live Demo

For the best demonstration of the full capabilities of the MRZ Scanner JavaScript Edition, please visit the [official Dynamsoft MRZ Scanner JavaScript demo](https://demo.dynamsoft.com/mrz-scanner/). Please note that this demo is more sophisticated than the Hello World implementation, so if you are curious about it, please visit the [Demo]({{ site.codegallery }}demo/index.html) section of the code gallery.

## Conclusion

As you can see, all it takes 6 lines (not counting the comments) to get the MRZ Scanner up and running!

The simplest implementation means that the MRZ Scanner is initialized with the default UI, which for all intents and purposes covers the majority of the features of the MRZ Scanner. In addition, since this is the simplest implementation, no action is taken once the user clicks "Done" in the result view.

For a further breakdown of the Hello World code, please visit the [User Guide]({{ site.guides }}mrz-scanner.html).

To learn how to customize the scanner to fit your needs, please visit [Customizing the MRZ Scanner]({{ site.guides }}mrz-scanner-customization.html).
