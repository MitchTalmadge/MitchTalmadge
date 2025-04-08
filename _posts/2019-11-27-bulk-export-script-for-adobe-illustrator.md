---
title: "bulk-export script for adobe illustrator"
author: "Mitch Talmadge"
date: 2019-11-27T02:04:49.805-0700
last_modified_at: 2019-11-27T02:04:49.805-0700
tags: ["illustrator","adobe","export","programming","design"]
description: "A little script to automagically export your drawing to PNG, JPG, and SVG in many sizes."
image:
  path: /assets/2019-11-27-bulk-export-script-for-adobe-illustrator/1*4LLTUjaBw4yMA8cFWtjEuQ.png
---

_Note: This was never tested on a Mac; it was created with Windows. your mileage may vary._

I create a lot of logos in Illustrator. The most painful part is always at the end, when I have to export my work to the many formats that life demands. In the past, I would export each size I needed one at a time, or try to make use of the frustrating “Export for Screens” feature. Enough was enough.

> The great thing about being an engineer is that if something isn’t exactly how you want it… you just make it exactly how you want it. 

So I did. Here’s a script that takes a logo like this:

![](/assets/images/2019-11-27-bulk-export-script-for-adobe-illustrator/1*vBGx3AaHp4qJJN2tP0gXgw.png)

… and exports it into something like this:

![](/assets/images/2019-11-27-bulk-export-script-for-adobe-illustrator/1*4LLTUjaBw4yMA8cFWtjEuQ.png)

More specifically, the resulting file structure looks like this:
```
Drawing.ai
JPG
  Drawing-16px.jpg
  Drawing-32px.jpg
  Drawing-50px.jpg
  Drawing-64px.jpg
  Drawing-100px.jpg
  Drawing-150px.jpg
  Drawing-256px.jpg
  Drawing-300px.jpg
  Drawing-512px.jpg
  Drawing-1024px.jpg
PNG
  Drawing-16px.png
  Drawing-32px.png
  Drawing-50px.png
  Drawing-64px.png
  Drawing-100px.png
  Drawing-150px.png
  Drawing-256px.png
  Drawing-300px.png
  Drawing-512px.png
  Drawing-1024px.png
SVG
  Drawing.svg
```

Nice and organized! I picked the dimensions I use most, but you can of course edit the script and pick the ones you need.

Convinced? Here’s the script and instructions:
## The Script
```jsx
#target Illustrator

/**
 * This script will export an Illustrator file into multiple sizes of multiple file types.
 * @author Mitch Talmadge ( https://MitchTalmadge.com )
 */

if (app.documents.length > 0) {
  main();
} else {
  Window.alert("Cancelled export.");
}

function main() {
  var sizes = [1024, 512, 300, 256, 150, 100, 64, 50, 32, 16];
  var document = app.activeDocument;
  var afile = document.fullName;
  var filename = afile.name.split('.')[0];

  var svgFolder = new Folder(afile.parent.fsName + "/SVG");
  if (!svgFolder.exists) {
    svgFolder.create();
  }

  var pngFolder = new Folder(afile.parent.fsName + "/PNG");
  if (!pngFolder.exists) {
    pngFolder.create();
  }

  var jpgFolder = new Folder(afile.parent.fsName + "/JPG");
  if (!jpgFolder.exists) {
    jpgFolder.create();
  }

  Window.alert("Press OK to begin exporting.");

  var size, file;

  if (svgFolder != null) {
    var options = new ExportOptionsSVG();
    options.cssProperties = SVGCSSPropertyLocation.PRESENTATIONATTRIBUTES;
    options.documentEncoding = SVGDocumentEncoding.UTF8;
    options.fontType = SVGFontType.OUTLINEFONT;
    options.fontSubsetting = SVGFontSubsetting.None;
    options.preserveEditability = false;
    options.embedRasterImages = true;

    file = new File(svgFolder.fsName + '/' + filename + ".svg");

    document.exportFile(file, ExportType.SVG, options);
  }

  if (pngFolder != null) {
    var options = new ExportOptionsPNG24();
    options.antiAliasing = false;
    options.transparency = true;
    options.artBoardClipping = true;

    for (var i = 0; i < sizes.length; i++) {
      size = sizes[i];

      file = new File(pngFolder.fsName + '/' + filename + "-" + size + "px.png");

      var scale = size / document.height;

      if (scale <= 7.76) {
        options.verticalScale = 100 * scale;
        options.horizontalScale = 100 * scale;

        document.exportFile(file, ExportType.PNG24, options);
      } else {
        Window.alert("Cannot scale to required size. Artboard too small.");
        reopenDocument(document, afile);
        return;
      }
    }
  }

  if (jpgFolder != null) {
    var options = new ExportOptionsJPEG();
    options.antiAliasing = false;
    options.qualitySetting = 100;
    options.optimization = true;
    options.artBoardClipping = true;

    for (var i = 0; i < sizes.length; i++) {
      size = sizes[i];

      file = new File(jpgFolder.fsName + '/' + filename + "-" + size + "px.jpg");

      var scale = size / document.height;

      if (scale <= 7.76) {
        options.verticalScale = 100 * scale;
        options.horizontalScale = 100 * scale;

        document.exportFile(file, ExportType.JPEG, options);
      } else {
        Window.alert("Cannot scale to required size. Artboard too small.");
        reopenDocument(document, afile);
        return;
      }
    }
  }

  Window.alert("Images have been exported!");

  reopenDocument(document, afile);
}

function reopenDocument(document, afile) {
  document.close(SaveOptions.DONOTSAVECHANGES);
  app.open(afile);
}
```

Save it as a jsx file, such as `Illustrator_Drawing_Exporter.jsx` .
## Usage / Installation

There are three ways to use the script.
1. With an Illustrator drawing already open and in-view, right click on the jsx file and open it with Illustrator. You will be prompted to begin exporting.
2. Inside Illustrator, with the drawing in-view, go to `File -> Scripts -> Other Script…` and open the jsx file from here.
3. To add the script to the `File -> Scripts` menu permanently, open the Illustrator install directory (you’ll have to find it yourself, it varies), and put the script inside `Presets/en_US/Scripts` . Now restart Illustrator and you can use the script at any time from the `File -> Scripts` menu.

Hope this is helpful, let me know if you have questions!

