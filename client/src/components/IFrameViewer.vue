<template>
    <input @click="search" value="Search text" type="button"/>
    <input @click="saveDoc" value="Save file" type="button"/>
    <input @click="loadDoc" value="Load prev file" type="button"/>
    <input @click="chgToPg3" value="Change to `Contents` Page" type="button"/>
    <div id="pageContainer">
        <iframe 
            id="pdf-js-viewer"
            :src="getPath"
            title="IFrameViewer"
            frameborder="0"
            width="1000"
            height="1200"
            >
        </iframe>
    </div>
</template>
  
<script>

export default {
    name: "PdfViewer",
    data() {
        return {
            docBlob: null,
            docPath: null,
            pathViewer: './src/public/pdfjs-dist/web/viewer.html',
            query: '?file=',
            pathFile: '../../pdf/sample.pdf',    //must be relative to `viewer.html` location
        }
    },
    computed:{
        getPath(){
            return this.pathViewer + this.query + this.pathFile
        }
    },
    methods: {
        search(){
            //search and highlight text
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            app.pdfViewer.eventBus.dispatch('find', {
                query: 'Sample PDF', 
                highlightAll: true
            })
        },
        async saveDoc(){
            //get document properties
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            const doc = await app.pdfDocument
            console.log(`Get numPages: ${doc.numPages}`)
            const blob = await doc.getData()
            this.docBlob = blob
            console.log(`Get Uint8Array (first 50...): ${blob.slice(0,50)}`)
        },
        async loadDoc(){
            //load document from typed array
            //expected workflow: click save doc, open a new document (manually) using `Open File` button, then click load doc 
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            await app.open({data: this.docBlob})
        },
        chgToPg3(){
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            app.page = 3
        }

    }
}
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