<template>
    <input @click="search" value="Search text" type="button"/>
    <input @click="saveDoc" value="Save file" type="button"/>
    <input @click="loadDoc" value="Load prev file" type="button"/>
    <input @click="chgToPg3" value="Change to `Contents` Page" type="button"/>
    <input @click="highlightText" value="Extract Text" type="button"/>
    <div id="pageContainer">
        <iframe
            ref="frame"
            @load="iframeLoaded" 
             
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
            pathFile: '../../pdf/sample.pdf#search=Simple',    //must be relative to `viewer.html` location
        }
    },
    computed:{
        getPath(){
            return this.pathViewer + this.query + this.pathFile
        }
    },
    methods: {
        // Listener Logic
        //listener and selector for user-selected text
        iframeLoaded() {
            const app = document.getElementById('pdf-js-viewer').contentWindow.document//.PDFViewerApplication
            app.addEventListener('selectionchange', () => {
                console.log(  this.getSelectedText()  )
            })
        },
        getSelectedText() {
            const iframeWindow = document.getElementById('pdf-js-viewer').contentWindow.document
            if (iframeWindow) {
                  return iframeWindow.getSelection().toString();
               } 
               else if (document.selection) {
                   return document.selection.createRange().text;
               }
               return '';
        },
        getSelectedImage(){
            const iframeWindow = document.getElementById('pdf-js-viewer').contentWindow.document
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            const pageIndex = app.page - 1
            const _page = app.pdfViewer._pages[pageIndex]
            _page.canvas.getContext('2d').getImageData(1,1,1,1)     //x, y, width, height

        },



        // Button Logic
        search(){
            //search and highlight text
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            app.pdfViewer.eventBus.dispatch('find', {
                query: 'Sample PDF', 
                highlightAll: true,
                caseSensitive: false,
                findPrevious: undefined,
                phraseSearch: true
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
            //app.zoomIn(5)
        },
        highlightText(){
            const selected = this.getSelectionCoords()
            this.showHighlight(selected)
            console.log(selected)
        
        },
        getSelectionCoords() {
            const iframeWindow = document.getElementById('pdf-js-viewer').contentWindow
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            const pageIndex = app.page - 1

            const _page = app.pdfViewer._pages[pageIndex]
            const pageRect = _page.canvas.getClientRects()[0]
            if(iframeWindow.getSelection().rangeCount == 0 ){
                console.log('ERROR: no text selected')
                return {}
            }
            if(iframeWindow.getSelection().rangeCount > 1 ){
                console.log('ERROR: only a single, continuous text may be selected')
                return {}
            }
            const selectionRects = iframeWindow.getSelection().getRangeAt(0).getClientRects()
            const viewport = _page.viewport
            const selectionRectsList = Object.values(selectionRects)
            const selected = selectionRectsList.map(function (r) {
                return viewport.convertToPdfPoint(r.left - pageRect.x, r.top - pageRect.y).concat(
                    viewport.convertToPdfPoint(r.right - pageRect.x, r.bottom - pageRect.y)); 
                })    // left, top, right, bottom
            return {pageIndex: pageIndex, coordinates: selected}      //only allow a single, continuous selection
        },
        showHighlight({pageIndex, coordinates}) {
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            const _page = app.pdfViewer._pages[pageIndex]
            const viewport = _page.viewport
            
            coordinates.forEach(function (rect) {
                let highlightColor = 'ff9900'   //generateColor()
                let bounds = viewport.convertToViewportRectangle(rect);

                var x1 = Math.min(bounds[0], bounds[2]);
                var y1 = Math.min(bounds[1], bounds[3]);
                var width =  Math.abs(bounds[0] - bounds[2]);
                var hight = Math.abs(bounds[1] - bounds[3]);
                
                var el = createRectDiv([x1, y1, width, hight], highlightColor);
                _page.textLayer.div.appendChild(el)
            }, this)




        }
    }
}


const generateColor = () => {
    return Math.floor(Math.random()*16777215).toString(16)
}

function createRectDiv(boundBox, highlightColor){
    // console.log(randomColor);
    var el = document.createElement('div');
    el.setAttribute('class', 'hiDiv')
    el.setAttribute('style', 'position: absolute; background-color: #'+highlightColor+'; opacity: 0.95;' + 
    'left:' + boundBox[0] + 'px; top:' + boundBox[1] + 'px;' +
    'width:' + boundBox[2] + 'px; height:' + boundBox[3] + 'px;');
    return el;
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