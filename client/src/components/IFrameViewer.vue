<template>
    <input @click="search" value="Search text" type="button"/>
    <input @click="test" value="TEST" type="button"/>
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