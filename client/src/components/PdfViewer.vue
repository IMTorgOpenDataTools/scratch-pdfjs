<template>

    <form name="uploadForm">
        <input id="selectInput" @change="selectInput" type="file" accept=".pdf" single />
        <br/>
        <label for="fileNum">Selected files:</label>
        <output id="fileNum">0</output>
        <br/>
      <input @click="getPdf" value="Upload file" type="button"/>
    </form>
    <input @click="search" value="Search text" type="button"/>

    <div id="pageContainer">
        <div id="viewer" class="pdfViewer"></div>
    </div>
</template>
  
<script>
import * as pdfjsLib from "pdfjs-dist/build/pdf"
import *  as pdfjsViewer from "pdfjs-dist/web/pdf_viewer"
import * as pdfWorker from "pdfjs-dist/build/pdf.worker.mjs"

// Setting worker path to worker bundle.
pdfjsLib.GlobalWorkerOptions.workerSrc = pdfWorker

export { pdfjsLib }
pdfjsLib.GlobalWorkerOptions.workerSrc = "https://cdn.jsdelivr.net/npm/pdfjs-dist@2.0.943/build/pdf.worker.min.js"

export { pdfjsViewer }
import "pdfjs-dist/web/pdf_viewer.css"




export default {
    name: "PdfViewer",
    //props: { docPath: String },
    data() {
        return {
            docBlob: null,
            docPath: "./src/public/pdf/compressed.tracemonkey-pldi-09.pdf",

            pdf: null,
            pdfViwer: null
        }
    },
    methods: {
        selectInput(){
            const selectInput = document.getElementById("selectInput")
            document.getElementById("fileNum").textContent = selectInput.files.length
            this.docBlob = null
            this.docPath = URL.createObjectURL(selectInput.files[0])
        },
        search(){
            //this.pdfViewer.eventBus.on('pagesinit', function(){this.pdfViewer.eventBus.dispatch('find', {query: 'trace'})  })
            //this.pdfViewer.eventBus.dispatch('find', {query: 'trace', highlightAll:true})
            // TODO: HOW TO DO THIS ^^^ ???
        },



        async getPdf() {
            let container = document.getElementById("pageContainer");


            // Args getDocument
            // Some PDFs need external cmaps.
            const CMAP_URL = "pdfjs-dist/cmaps/"
            const CMAP_PACKED = true
            const SANDBOX_BUNDLE_SRC = new URL("pdfjs-dist/build/pdf.sandbox.mjs",
                window.location
            )
            const ENABLE_XFA = true
            const SEARCH_FOR = "Trace";     //test term


            // Args PdfViewer
            const eventBus = new pdfjsViewer.EventBus();
            // (Optionally) enable hyperlinks within PDF files.
            const pdfLinkService = new pdfjsViewer.PDFLinkService({
                eventBus,
            });
            // (Optionally) enable find controller.
            const pdfFindController = new pdfjsViewer.PDFFindController({
                eventBus,
                linkService: pdfLinkService,
            });
            // (Optionally) enable scripting support.
            const pdfScriptingManager = new pdfjsViewer.PDFScriptingManager({
                eventBus,
                sandboxBundleSrc: SANDBOX_BUNDLE_SRC,
            });



            let pdfViewer = new pdfjsViewer.PDFSinglePageViewer({     //new pdfjsViewer.PDFViewer({
                container,
                eventBus,

                linkService: pdfLinkService,
                findController: pdfFindController,
                scriptingManager: pdfScriptingManager,
            });
            pdfLinkService.setViewer(pdfViewer)
            pdfScriptingManager.setViewer(pdfViewer)
            eventBus.on("pagesinit", function () {
              // We can use pdfViewer now, e.g. let's change default scale.
              pdfViewer.currentScaleValue = "page-width";
              // We can try searching for things.
              if (SEARCH_FOR) {
                eventBus.dispatch("find", { type: "", query: SEARCH_FOR }, );
              }
            });



            let pdf = await pdfjsLib.getDocument({
                url: this.docPath,
                cMapUrl: CMAP_URL,
                cMapPacked: CMAP_PACKED,
                enableXfa: ENABLE_XFA,
            }).promise.then(function(pdf){
                console.log(`pdf loaded: ${pdf}`)
                return pdf
            }, function (reason) {
                console.error(`error from: ${reason}`)
            })
            let blob = await pdf.getData()
            const docInitParams = {data: blob}
            this.docBlob = docInitParams
            console.log(`Save pdf object ('Uint8Array') to storage: ${this.docBlob}...`)
            console.log(`...then read back the object, later: ${blob}`)
            pdfjsLib.getDocument(docInitParams).promise.then(function(pdfFromRaw){
                
                const numPages = pdfFromRaw.numPages
                console.log(`Saved document ('Uint8Array') STILL contains ${numPages} pages from saved `)
            })

            console.log(`PDF Document proxy: ${pdf}`)
            console.log(`Document contains ${pdf.numPages} pages`)
            const pageNumber = 2
            pdf.getPage(pageNumber).then(function(page) {
                console.log(`Page loaded: ${page}`)
            })

            await pdfViewer.setDocument(pdf)
        },
    },
};
</script>
  
<style>
#pageContainer {
    position: absolute;
    /*
    overflow: auto;
    
    margin: auto;
    width: 80%;*/
}

/*
div.page {
    display: inline-block;
}*/
</style>