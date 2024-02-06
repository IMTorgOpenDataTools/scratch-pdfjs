<template>
    <input @click="search" value="Search text" type="button"/>
    <input @click="saveDoc" value="Save file" type="button"/>
    <input @click="loadDoc" value="Load prev file" type="button"/>
    <input @click="chgToPg3" value="Change to `Contents` Page" type="button"/>
    <br/>
    <input @click="extractTextRadial" :value="'Select and Extract Text (' + this.extractText +')'" type="button"/>
    <input @click="highlightText" value="Hightlight Selected Text" type="button"/>
    <br/>
    <input @click="extractImageRadial" :value="'Select and Extract Image (' + this.extractImage +')'" type="button"/>
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

            extractText: false,
            extractImage: false
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
            const app = document.getElementById('pdf-js-viewer').contentWindow.document
        },
        extractTextRadial(){
            const app = document.getElementById('pdf-js-viewer').contentWindow.document
            this.extractText = !this.extractText
            if(this.extractText){
                app.addEventListener('selectionchange', this.logText)
            }else{
                app.removeEventListener('selectionchange', this.logText)
            }
        },
        logText(e){
            console.log(this.getSelectedText(e))
        },
        getSelectedText(e) {
            const iframeWindow = document.getElementById('pdf-js-viewer').contentWindow.document
            if (iframeWindow) {
                  return iframeWindow.getSelection().toString();
               } 
               else if (document.selection) {
                   return document.selection.createRange().text;
               }
               return '';
        },
        extractImageRadial(){
            // TODO: this fails !!!
            //google: pdfjs pdfviewer area selector
            //ref: https://stackoverflow.com/questions/17703906/use-a-canvas-to-select-a-portion-of-a-pdf-document?rq=3
            //ref: https://stackoverflow.com/questions/49605369/highlight-area-in-pdfjs-canvas-based-on-native-pdf-coordinates
            //ref: https://stackoverflow.com/questions/55411493/pdf-js-with-canvas-draw-draw-rectangle-in-canvas-with-pdf-loaded-using-pdf-js?noredirect=1&lq=1
            //ref: https://stackoverflow.com/questions/58590845/draw-rectangle-in-canvas-with-loaded-pdf-file-using-pdf-js?noredirect=1&lq=1
            this.extractImage = !this.extractImage

            const iframeWindow = document.getElementById('pdf-js-viewer').contentWindow
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            const pageIndex = app.page - 1

            const _page = app.pdfViewer._pages[pageIndex]
            const canvas = _page.canvas.getContext("2d")
            //var x1 = 0, y1 = 0, x2 = 0, y2 = 0
            if(this.extractImage){
                iframeWindow.addEventListener('onmousedown', this.onmousechange)
                iframeWindow.addEventListener('onmouseup', this.onmousechange)
            }else{
                iframeWindow.removeEventListener('onmousedown', this.onmousechange)
                iframeWindow.removeEventListener('onmouseup', this.onmousechange)
            }
        },

            
        onmousechange(e) {
                //div.hidden = 0;
                const tmp = window.getSelection().toString()
                console.log(tmp)
                let x1 = 0
                let y1 = 0
                let x2 = e.clientX;
                let y2 = e.clientY;
                console.log(`x1:${x1},y1: ${y1}    x2:${x2},y2: ${y2}`)
                //canvas.removeEventListener('onmousedown', (e) => {onmousedown(e)})
                //reCalc();
                /*
            }
            onmousemove = function(e) {
                x2 = e.clientX;
                y2 = e.clientY;
                console.log(`x1:${x1},y1: ${y1}    x2:${x2},y2: ${y2}`)
                canvas .removeEventListener('onmousemove', (e) => {onmousemove(e)})
                //reCalc();
            }*
            function onmouseup(e) {
                //div.hidden = 1;
                console.log(`x1:${x1},y1: ${y1}    x2:${x2},y2: ${y2}`)
                canvas.removeEventListener('onmouseup', (e) => {onmouseup(e)})
            }*/

        },



        // Button Logic
        search(){
            //search and highlight text
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            app.pdfViewer.eventBus.dispatch('find', {
                query: '1999', 
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



        // Logic
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
        },

        getExtractedImage(){
            const app = document.getElementById('pdf-js-viewer').contentWindow.document//.PDFViewerApplication

        },
        getSelectedImage(){
            //ref: https://stackoverflow.com/questions/13416800/how-to-generate-an-image-from-imagedata-in-javascript
            //ref: https://stackoverflow.com/questions/923885/capture-html-canvas-as-gif-jpg-png-pdf?noredirect=1&lq=1
            const iframeWindow = document.getElementById('pdf-js-viewer').contentWindow.document
            const app = document.getElementById('pdf-js-viewer').contentWindow.PDFViewerApplication
            const pageIndex = app.page - 1
            const _page = app.pdfViewer._pages[pageIndex]
            _page.canvas.getContext('2d').getImageData(1,1,1,1)     //x, y, width, height

        },
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
}

#div {
    border: 1px dotted #000;
    position: absolute;
}

/*
#cloud_main_page{
  width:400px;
  height:400px;
  
  position:relative;
}
#pageContainer #pdf-js-viewer{	
	position:absolute;
	
	background-color:rgba(6, 217, 160, 0.05);
	border: 1px solid rgba(6, 217, 160, 0.3);	
}*/
</style>