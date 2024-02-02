


# Pdfjs Steps

These sections, based on approach used by component, list steps (saved in each commit) increasing in functionality sophistication.

### `PdfViewer.vue`

Uses the node package, directly, with `npm install pdfjs-dist`.

* `7ab37bd` Basic setup with Viewer
* `56fd043` Document file upload: adds error handling
* `b08cee2` Save, then Load from ('Uint8Array'): as if from backend
* `0d8faaa` Extract all text


### `IFrameViewer.vue`

Uses the [latest stable release](https://mozilla.github.io/pdf.js/getting_started/#download), which is unzipped into the `public/pdfjs-dist/` folder.  Here, and `<iframe>` sources the `public/pdfjs-dist/web/viewer.html` file.

* `fa70a6e` Basic setup with `IFrameViewer`
* `9832d80` Search for text in Viewer and highlight the text
* `fb33eef` Customize Viewer tool bar with `customToolbar.js`
* `128b934` Save, then Load from ('Uint8Array'): as if from backend
* `aee0da1` Move to selected page and zoom
* Extract manually selected text
* Extract manually selected image / diagram
* Highlight parts of the document
* Improve whole document text extraction





# Format code

VsCode: `Shift + Alt + F`

Prettier: `CMD + SHIFT + P`



# References

### Setup 

* [vue setup - old](https://stackoverflow.com/questions/65750584/how-to-import-mozilla-pdf-js-in-vue-project)
* [mozilla examples](https://github.com/mozilla/pdf.js/blob/master/examples/learning/helloworld.html)

### Foundations

* [How PDF.js Works](https://pdfjs.express/blog/how-pdf-js-works)
* [How to Use PDF.js in 2 Easy Steps](https://pdfjs.express/blog/how-to-use-pdf-js)
* [Mediathread: use and extend PDF.js Viewer](https://www.columbia.edu/~njn2118/journal/2021/7/30.html)
* Extract text from pdf
  - [ref-1](https://stackoverflow.com/questions/1554280/how-to-extract-text-from-a-pdf-in-javascript)
  - [ref-2](https://stackoverflow.com/questions/40635979/how-to-correctly-extract-text-from-a-pdf-using-pdf-js)

### Viewer functions
* [Pdf.js and viewer.js. Pass a stream or blob to the viewer](https://stackoverflow.com/questions/24535799/pdf-js-and-viewer-js-pass-a-stream-or-blob-to-the-viewer)
* [How do I retrieve text from user selection in pdf.js?](https://stackoverflow.com/questions/48950038/how-do-i-retrieve-text-from-user-selection-in-pdf-js)
* [Get highlighted coordinates and display highlight](https://gist.github.com/yurydelendik/f2b846dae7cb29c86d23)
* [PDF.js - Using search function on embedded PDF](https://stackoverflow.com/questions/29987478/pdf-js-using-search-function-on-embedded-pdf)