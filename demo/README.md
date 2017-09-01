# // Barcode Writer in Pure JavaScript


<a href="http://metafloor.github.io/bwip-js"><img alt="bwip-js" align="right" src="http://metafloor.github.io/bwip-js/images/bwip-js.png"></a>
bwip-js is a translation to native JavaScript of the amazing code provided in [Barcode Writer in Pure PostScript](https://github.com/bwipp/postscriptbarcode).  The translated code can run on any modern browser or JavaScript-based server framework.

The software has encoding modules for over 90 different barcode types and standards.
All linear and two-dimensional barcodes in common use (and many uncommon
ones) are available.  An exhaustive list of supported barcode types can be
found at the end of this document.

> Version 1.5 has a replacement for the enscriptem-compiled FreeType library (which was causing 
> issues with popular frameworks and pre-allocated too much memory for use in embedded servers).
> It *is* still possible to use FreeType with Node.js, but you must explicitly enable it.
> By default, you get the replacement.  Browsers only use the replacement.
>
> The new font library uses bitmapped fonts that were generated by freetype.js at the integral
> scale factors of 1x - 9x.  There should be no change in how the fonts display
> when the default font sizes are used.  If you use custom sizes *OR* use non-uniform scaling,
> there may be noticeable differences.
>
> See [FreeType Replacement](https://github.com/metafloor/bwip-js/wiki/FreeType-Replacement) for more details.

## Status 

* Current bwip-js version is 1.5.0 (2017-09-01)
* Current BWIPP version is 2017-06-09
* Node.js compatibility >= v0.10

## Links

* [Home Page](http://metafloor.github.io/bwip-js/)
* [Repository](https://github.com/metafloor/bwip-js)
* [Online Barcode Generator](http://metafloor.github.io/bwip-js/demo/demo.html)
* [Online Barcode API](https://github.com/metafloor/bwip-js/wiki/Online-Barcode-API)
* [Node.js npm Page](https://www.npmjs.com/package/bwip-js)
* [BWIPP Documentation](https://github.com/bwipp/postscriptbarcode/wiki)
* [Differences between BWIPP and bwipjs](https://github.com/metafloor/bwip-js/wiki/Differences-between-BWIPP-and-bwipjs)
* [Supported Barcode Types](https://github.com/metafloor/bwip-js/wiki/BWIPP-Barcode-Types)

## Online Barcode Generator

An [online barcode generator](http://metafloor.github.io/bwip-js/demo/demo.html)
demonstrates all of the features of bwip-js.  As of version 1.5, the FreeType
library is no longer supported in the demo.  Only the OCR-A and OCR-B fonts are 
available.

The demo is tested on the latest versions of Firefox and Chrome, along with IE10 and IE11.
Microsoft Edge should work, and so should the latest versions of Opera and Safari,
but they are untested.

## Online Barcode API

A bwip-js barcode service is available online, ready to serve up barcode images
on demand.

You can generate barcodes from anywhere on the web.  Embed the URLs in your
HTML documents or retrieve the barcode images directly from your non-JavaScript
server.  (JavaScript-based servers should use the bwip-js code directly - it will
be a lot more performant.)

For details on how to use this service, see [Online Barcode API](https://github.com/metafloor/bwip-js/wiki/Online-Barcode-API).

## Browser Usage

The following is a minimal example of using bwip-js in a React app.
It is based on the default `App.js` file generated by `create-react-app`.

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import bwipjs from 'bwip-js';

class App extends Component {
  constructor(props) {
	super(props);

    bwipjs('mycanvas', {
            bcid:        'code128',       // Barcode type
            text:        '0123456789',    // Text to encode
            scale:       3,               // 3x scaling factor
            height:      10,              // Bar height, in millimeters
            includetext: true,            // Show human-readable text
            textxalign:  'center',        // Always good to set this
        }, function (err, cvs) {
            if (err) {
                // Decide how to handle the error
                // `err` may be a string or Error object
            } else {
				// Nothing else to do in this example...
            }
        });
  }
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
		<canvas id="mycanvas"></canvas>
      </div>
    );
  }
}
export default App;
```

The `bwipjs()` method takes three parameters:

* The canvas on which to render the barcode.  This can by an `id` string or the actual
  canvas element.  The rendering will automatically resize the canvas to match the
  barcode image.
* A bwip-js/BWIPP options object.  See the Node.js Image Generator section below for a 
  description of this object.
* A callback to invoke once the canvas has been rendered.  It uses the traditional two
  parameter callback convention.  If `err` is set, `err` will contain an Error object
  or string.  Otherwise, `err` will be falsy and `cvs` will contain the canvas.
  This will be the same value passed in originally, either the `id` string or element.

If you would prefer to display the barcode using an `<img>` tag or with CSS `background-image`,
pass in a detached or hidden canvas to bwip-js, and use the canvas method
[HTMLCanvasElement.toDataURL](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL)
to get a data URL. For example:

```javascript
let canvas = document.createElement('canvas');
bwipjs(canvas, options, function(err, cvs) {
	if (err) {
		// handle the error
	} else {
		// Don't need the second param since we have the canvas in scope...	
		document.getElementById(myimg).src = canvas.toDataURL('image/png');
	}
});
```

If you want to generate barcodes with text, you *must* do one extra step after installing
bwip-js.  The browser-based font manager uses XHR to demand-load fonts as needed.  The
problem is that the fonts directory where bwip-js gets installed (under `node_modules`) is
usually not accessible.  Therefore, you must make the fonts visible to your applcation.

For React apps, you must copy (or link) the fonts to the `public/` directory.  If you
are using linux or unix-like:

    cd my-app
	ln -s node_modules/bwip-js/fonts public/bwipjs-fonts

Where `my-app` is the root of your application directory (typically created by `create-react-app`.  Under that directory, should be the `public/` and `node_modules/` directories.

If you are running windows, copy the files:

	cd my-app
	mkdir public\bwipjs-fonts
	copy node_modules\bwip-js\fonts\* public\bwipjs-fonts

For other frameworks, you may need to take additional steps.  The font manager assumes
there is a global variable called `process.env.PUBLIC_URL`, which is set automatically
by React.  If not set by your framework, assign it manually (adapt as necessary):

```javascript
window.process = { env: { PUBLIC_URL: path } };
```

Where `path` is the URL-path to the directory containing `bwipjs-fonts`.
The font manager appends `/bwipjs-fonts/file-name` to the value when it requests a file.

## Node.js Request Handler

The online barcode API is implemented as a Node.js application.
See the [Online Barcode API](https://github.com/metafloor/bwip-js/wiki/Online-Barcode-API) for details on how the URL query parameters must be structured.

A working, minimal example of how to use the request handler can be found in
`server.js`:

```javascript
// Simple HTTP server that renders barcode images using bwip-js.
const http   = require('http');
const bwipjs = require('bwip-js');

http.createServer(function(req, res) {
    // If the url does not begin /?bcid= then 404.  Otherwise, we end up
    // returning 400 on requests like favicon.ico.
    if (req.url.indexOf('/?bcid=') != 0) {
        res.writeHead(404, { 'Content-Type':'text/plain' });
        res.end('BWIPJS: Unknown request format.', 'utf8');
    } else {
        bwipjs(req, res);
    }

}).listen(3030);
```

If you run the above code on your local machine, you can test with the following URL:

```
http://localhost:3030/?bcid=isbn&text=978-1-56581-231-4+52250&includetext&guardwhitespace
```

The bwip-js request handler only operates on the URL query parameters and
ignores all path information.  Your application is free to structure the URL
path as needed to implement the desired HTTP request routing.

## Node.js Image Generator

You can also use bwip-js to generate PNG images directly.

```javascript
const bwipjs = require('bwip-js');

bwipjs.toBuffer({
        bcid:        'code128',       // Barcode type
        text:        '0123456789',    // Text to encode
        scale:       3,               // 3x scaling factor
        height:      10,              // Bar height, in millimeters
        includetext: true,            // Show human-readable text
        textxalign:  'center',        // Always good to set this
    }, function (err, png) {
        if (err) {
            // Decide how to handle the error
            // `err` may be a string or Error object
        } else {
            // `png` is a Buffer
            // png.length           : PNG file length
            // png.readUInt32BE(16) : PNG image width
            // png.readUInt32BE(20) : PNG image height
        }
    });
```

Only the first two options `bcid` and `text` are required.  The other bwip-js
specific options are:

- `scaleX` : The x-axis scaling factor.  Must be an integer > 0.  Default is 2.
- `scaleY` : The y-axis scaling factor.  Must be an integer > 0.  Default is `scaleX`.
- `scale` : Sets both the x-axis and y-axis scaling factors.  Must be an integer > 0.

- `rotate` : Allows rotating the image to one of the four orthogonal orientations.  A string value.  Must be one of:

    * `"N"` : Normal (not rotated).  The default.
    * `"R"` : Clockwise (right) 90 degree rotation.
    * `"L"` : Counter-clockwise (left) 90 degree rotation.
    * `"I"` : Inverted 180 degree rotation.

- `paddingwidth` : Sets the left and right padding (in points/pixels) around the rendered barcode.  Rotates and scales with the image.
- `paddingheight` : Sets the top and bottom padding (in points/pixels) around the rendered barcode.  Rotates and scales with the image.
- `monochrome` : Sets the human-readable text to render in monochrome.  Boolean `true` or `false`.  Default is `false` which renders 256-level gray-scale anti-aliased text.

All other options are BWIPP specific. You will need to consult the
[BWIPP documentation](https://github.com/bwipp/postscriptbarcode/wiki)
to determine what options are available for each barcode type.

Note that bwip-js normalizes the BWIPP `width` and `height` options to always be in millimeters.
The resulting images are rendered at 72 dpi.  To convert to pixels, use a factor of 2.835 px/mm
(72 dpi / 25.4 mm/in).  The bwip-js option `scale` multiplies both the `width` and `height`
options.  Likewise, the discrete scaling factors `scaleX` and `scaleY` multiply the `width`
and `height` options, respectively.

## Command Line Interface

bwip-js can be used as a command line tool.

```
npm install -g bwip-js
bwip-js --help
```

Usage example:

```
bwip-js --bcid=qrcode --text=123456789 ~/qrcode.png
```

## Installation

You can download the latest npm module using:

```
npm install bwip-js
```

Or the latest code from github:

	https://github.com/metafloor/bwip-js

(The bwip-js master branch and the npm version are kept sync'd.)

The software is organized as follows:

    bwip-js/
        barcode.ps        # The BWIPP PostScript barcode library
		browser-bitmap.js # Bitmap interface used by browsers.
		browser-bwipjs.js # The primary browser import.
		browser-fonts.js  # Browser-based font manager 
        bwipjs.js         # Main bwip-js code, cross-platform
        bwipp.js          # The cross-compiled BWIPP code
        demo.html         # The bwip-js demo
        freetype.js       # The Emscripten-compiled FreeType library
        node-bwipjs.js    # Primary node.js module
        node-bitmap.js    # Node.js module that implements a PNG encoder
		node-fonts.js     # Replacement for freetype.js
        server.js         # Node.js example server
        fonts/            # Font files
        lib/              # Files required by the demo

The above files are part of the *master* branch.  If you wish to 
compile bwip-js on your own, you will need to clone the *develop* branch, 
which contains the cross-compiler, test framework, code-coverage files,
benchmark framework, image proofs, etc.  Everything used to create and
validate bwip-js.

For details on how to compile and test bwip-js, see [Compiling bwipjs](https://github.com/metafloor/bwip-js/wiki/Compiling-bwipjs).


## Demo Usage

To run the demo from your HTTP server, you should link the `bwip-js` directory
to the server's public document directory and modify the server's configuration
files, if necessary.  Then navigate your browser to `bwip-js/demo.html`.

> You cannot run the demo using a `file://` URL.  The freetype library and the
> freetype replacement library use XHR, which must run over HTTP.

If you would like to implement your own interface to bwip-js, see [Integrating With Your Code](https://github.com/metafloor/bwip-js/wiki/Integrating-With-Your-Code).
You should also look at the `node-bwipjs.js` module to see how it was done for Node.js.

## Supported Barcode Types

&#x2022; auspost : AusPost 4 State Customer Code &#x2022; azteccode : Aztec Code &#x2022; azteccodecompact : Compact Aztec Code &#x2022; aztecrune : Aztec Runes &#x2022; bc412 : BC412 &#x2022; channelcode : Channel Code &#x2022; codablockf : Codablock F &#x2022; code11 : Code 11 &#x2022; code128 : Code 128 &#x2022; code16k : Code 16K &#x2022; code2of5 : Code 25 &#x2022; code32 : Italian Pharmacode &#x2022; code39 : Code 39 &#x2022; code39ext : Code 39 Extended &#x2022; code49 : Code 49 &#x2022; code93 : Code 93 &#x2022; code93ext : Code 93 Extended &#x2022; codeone : Code One &#x2022; coop2of5 : COOP 2 of 5 &#x2022; daft : Custom 4 state symbology &#x2022; databarexpanded : GS1 DataBar Expanded &#x2022; databarexpandedcomposite : GS1 DataBar Expanded Composite &#x2022; databarexpandedstacked : GS1 DataBar Expanded Stacked &#x2022; databarexpandedstackedcomposite : GS1 DataBar Expanded Stacked Composite &#x2022; databarlimited : GS1 DataBar Limited &#x2022; databarlimitedcomposite : GS1 DataBar Limited Composite &#x2022; databaromni : GS1 DataBar Omnidirectional &#x2022; databaromnicomposite : GS1 DataBar Omnidirectional Composite &#x2022; databarstacked : GS1 DataBar Stacked &#x2022; databarstackedcomposite : GS1 DataBar Stacked Composite &#x2022; databarstackedomni : GS1 DataBar Stacked Omnidirectional &#x2022; databarstackedomnicomposite : GS1 DataBar Stacked Omnidirectional Composite &#x2022; databartruncated : GS1 DataBar Truncated &#x2022; databartruncatedcomposite : GS1 DataBar Truncated Composite &#x2022; datalogic2of5 : Datalogic 2 of 5 &#x2022; datamatrix : Data Matrix &#x2022; datamatrixrectangular : Data Matrix Rectangular &#x2022; ean13 : EAN-13 &#x2022; ean13composite : EAN-13 Composite &#x2022; ean14 : GS1-14 &#x2022; ean2 : EAN-2 (2 digit addon) &#x2022; ean5 : EAN-5 (5 digit addon) &#x2022; ean8 : EAN-8 &#x2022; ean8composite : EAN-8 Composite &#x2022; flattermarken : Flattermarken &#x2022; gs1-128 : GS1-128 &#x2022; gs1-128composite : GS1-128 Composite &#x2022; gs1-cc : GS1 Composite 2D Component &#x2022; gs1datamatrix : GS1 Data Matrix &#x2022; gs1datamatrixrectangular : GS1 Data Matrix Rectangular &#x2022; gs1northamericancoupon : GS1 North American Coupon &#x2022; gs1qrcode : GS1 QR Code &#x2022; hanxin : Han Xin Code &#x2022; hibcazteccode : HIBC Aztec Code &#x2022; hibccodablockf : HIBC Codablock F &#x2022; hibccode128 : HIBC Code 128 &#x2022; hibccode39 : HIBC Code 39 &#x2022; hibcdatamatrix : HIBC Data Matrix &#x2022; hibcdatamatrixrectangular : HIBC Data Matrix Rectangular &#x2022; hibcmicropdf417 : HIBC MicroPDF417 &#x2022; hibcpdf417 : HIBC PDF417 &#x2022; hibcqrcode : HIBC QR Code &#x2022; iata2of5 : IATA 2 of 5 &#x2022; identcode : Deutsche Post Identcode &#x2022; industrial2of5 : Industrial 2 of 5 &#x2022; interleaved2of5 : Interleaved 2 of 5 (ITF) &#x2022; isbn : ISBN &#x2022; ismn : ISMN &#x2022; issn : ISSN &#x2022; itf14 : ITF-14 &#x2022; japanpost : Japan Post 4 State Customer Code &#x2022; kix : Royal Dutch TPG Post KIX &#x2022; leitcode : Deutsche Post Leitcode &#x2022; matrix2of5 : Matrix 2 of 5 &#x2022; maxicode : MaxiCode &#x2022; micropdf417 : MicroPDF417 &#x2022; microqrcode : Micro QR Code &#x2022; msi : MSI Modified Plessey &#x2022; onecode : USPS Intelligent Mail &#x2022; pdf417 : PDF417 &#x2022; pdf417compact : Compact PDF417 &#x2022; pharmacode : Pharmaceutical Binary Code &#x2022; pharmacode2 : Two-track Pharmacode &#x2022; planet : USPS PLANET &#x2022; plessey : Plessey UK &#x2022; posicode : PosiCode &#x2022; postnet : USPS POSTNET &#x2022; pzn : Pharmazentralnummer (PZN) &#x2022; qrcode : QR Code &#x2022; rationalizedCodabar : Codabar &#x2022; raw : Custom 1D symbology &#x2022; royalmail : Royal Mail 4 State Customer Code &#x2022; sscc18 : SSCC-18 &#x2022; symbol : Miscellaneous symbols &#x2022; telepen : Telepen &#x2022; telepennumeric : Telepen Numeric &#x2022; upca : UPC-A &#x2022; upcacomposite : UPC-A Composite &#x2022; upce : UPC-E &#x2022; upcecomposite : UPC-E Composite &#x2022;\n