<template>
    <div>
        <!--<button @click="customStart">customStart</button>
        <button @click="customParticipate">customParticipate</button>
        <button @click="customEnd">customEnd</button>-->
        <form class="container" action="#" method="post" accept-charset="utf-8">
            <div v-if="showScanner" id="videoCapture" ref="videoCapture"></div>
            <input v-if="captureObject" @click="beginScanning" id="scan" ref="scan" type="button" class="button is-primary pt-main-button" value="Start scanning">
            <input v-else id="selectorElement" type="file" accept="image/*" value="Select an image" class="is-invisible is-primary pt-main-button" name="ir">
        </form>	
        <div class="loader spinner is-invisible" id="spinner" ref="spinner"></div>
    </div>
</template>

<script>
import CraftAr from '../utils/craftar.min'

export default {
    methods: {
    },
    data:()=>({
        finderResults: false,
        cloudRecognition: null,
        captureObject: null,
        showScanner: true,
        capturerInitialized: false
    }),
    created(){
        this.cloudRecognition =  new craftar.CloudRecognition({
            token: '4d1c9e52d5e24197'
        });
        
    },
    mounted(){
        var scanButton = document.getElementById('scan')

        if ( craftar.supportsCapture() ){
            let that = this
            this.setupCapture(( err, captureObject ) => {

                if ( err ){
                    // Capture setup failed (user rejected to open the camera)
                    // switch to selector mode
                    that.switchToSelector()
                } else {
                    this.captureObject = captureObject        
                }

            });

        }else if( craftar.supportsImageSelector() ){
            // Capture not supported, switch to selector mode
            this.switchToSelector();
        }else{
            alert("This browser doesn't support HTML5 features needed for CraftAR HTML5 Library");
        }
    },
    methods:{
        beginScanning(){
            this.showScanner = true
            var captureDivElement = document.getElementById( 'videoCapture' )
            captureDivElement.appendChild(this.captureObject.domElement)
            if (!this.capturerInitialized){
                this.setCaptureResultsListener()
                this.capturerInitialized = true
            }
            var spinnerElement = this.$refs.spinner
            spinnerElement.setAttribute("class", "loader spinner")
            this.finderResults = false
            this.cloudRecognition.startFinder(this.captureObject, 2000, 3)
        },
        scanAction(){
            spinnerElement.setAttribute("class", "loader spinner")
            this.finderResults = false
            this.cloudRecognition.startFinder(this.captureObject, 2000, 3)
        },
        switchToSelector() {
            var scanButton = document.getElementById('scan');
            var selectorElement = document.getElementById('selectorElement');
            var spinnerElement = document.getElementById('spinner');

            $("#selectorElement").prettyfile({
                html: "Choose an image"
            });

            scanButton.setAttribute("class", "is-invisible");
            selectorElement.removeAttribute("class", "is-invisible");


            var selector = new craftar.ImageSelector(selectorElement);
            selector.addListener('image', (craftarImage) => {
                spinnerElement.setAttribute("class", "loader spinner");
                this.cloudRecognition.search(craftarImage);
            });
            {{debugger}}
            this.setSelectorResultsListener();
        },

        //window.addEventListener("load", init, false);

        setupCapture( callback ){

            var capture = new craftar.Capture();

            capture.addListener('started', function(){

                callback( null, capture );

            });

            capture.addListener('error', function( error ){

                callback( error, capture );

            });

            capture.start();

        },

        setSelectorResultsListener() {
            this.cloudRecognition.addListener('results', function(error, results, xhr) {
                var spinnerElement = document.getElementById('spinner');
                spinnerElement.setAttribute("class", "loader spinner is-invisible");

                if (results.results && results.results.length > 0) {
                    this.renderResults( results );
                } else {
                    alert("No results found, select an image that contains an object to scan.");
                }
            });
        },

        setCaptureResultsListener() {
            this.cloudRecognition.addListener('results', (err, results, xhr) => {

                if (results.results && results.results.length > 0) {
                    this.finderResults = true;
                    this.renderResults( results );
                    this.cloudRecognition.stopFinder();
                    var spinnerElement = document.getElementById('spinner');
                    spinnerElement.setAttribute("class", "loader spinner is-invisible");
                }
            });

            this.cloudRecognition.addListener('finderFinished', () => {
                var spinnerElement = document.getElementById('spinner');
                spinnerElement.setAttribute("class", "loader spinner is-invisible");
                if (!this.finderResults) {
                    alert("No results found, point to an object.");
                    this.showScanner = false
                }
            });
        },
        renderResults( results ){
            
            //It is called once an item is recognized

            // results sample: {"search_time":87,"results":[{"item":{"name":"PET_Fantal_500ml","url":"promos.q.orchextra.io/test-guillermo-q","custom":"https://crs-beta-catchoom.s3.amazonaws.com/collections/a2a7e94186ef44b3be3f7dbf0ce70b49/metadata/592bbeb24ab446de8b6d2da17a2230ff.json","content":null,"trackable":false,"uuid":"592bbeb24ab446de8b6d2da17a2230ff"},"image":{"thumb_120":"https://crs-beta-catchoom.s3.amazonaws.com/cache/collections/a2a7e94186ef44b3be3f7dbf0ce70b49/images/592bbeb24ab446de8b6d2da17a2230ff_85371219b9ef4bc88e446a1b18d47764_thumb_120.jpe","uuid":"85371219b9ef4bc88e446a1b18d47764"},"score":59}]}
            var resultItem = results.results[0];

            /*var template = document.getElementById("resultTemplate");
            var resultsElement = document.getElementById( 'resultList' );
            var spinnerElement = document.getElementById('spinner');

            var itemHTML =  Handlebars.compile(template.innerHTML);

            var resultEl = document.createElement('div');
            resultEl.innerHTML = itemHTML({thumbnailUrl: resultItem.image.thumb_120, itemUrl: resultItem.item.url, itemName: resultItem.item.name});
            */


            /*var resultsElement = document.getElementById( 'resultList' );
            var resultEl = document.createElement('div');
            resultEl.innerHTML = '<recognized-element thumbnail-url="'+resultItem.image.thumb_120+'"  \
            item-name="'+resultItem.item.name+'" \
            item-url="'+resultItem.item.url+'"></recognized-element>'

            resultsElement.insertBefore(resultEl, resultsElement.firstChild);*/
            this.$emit('input', results.results)
            debugger;
            this.$parent.recognizedItems = Array.from(results.results)
            this.showScanner = false
        }
    }
}
</script>

<style>
#spinner {
    position: absolute;
    top: 15px;
    left: 10px;
    width: 20px;
    height: 0px;
}
</style>