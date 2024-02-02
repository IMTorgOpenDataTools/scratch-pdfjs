


# Pdfjs Steps

These sections, based on approach used by component, list steps (saved in each commit) increasing in functionality sophistication.

### `PdfViewer.vue`

Uses the node package, directly, with `npm install pdfjs-dist`.

* `7ab37bd` Basic setup with Viewer
* `56fd043` Document file upload: adds error handling
* `b08cee2` Save, then Load from ('Uint8Array'): as if from backend


### `IFrameViewer.vue`

Uses the [latest stable release](https://mozilla.github.io/pdf.js/getting_started/#download), which is unzipped into the `public/pdfjs-dist/` folder.  Here, and `<iframe>` sources the `public/pdfjs-dist/web/viewer.html` file.

* `fa70a6e` Basic setup with `IFrameViewer`
* `9832d80` Search for text in Viewer and highlight the text
* `fb33eef` Customize Viewer tool bar with `customToolbar.js`
* Save, then Load from ('Uint8Array'): as if from backend
* Extract selected text
* Extract selected image / diagram
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
* 