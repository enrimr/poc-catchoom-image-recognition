<template>
    <div>
        <!--<button @click="customStart">customStart</button>
        <button @click="customParticipate">customParticipate</button>
        <button @click="customEnd">customEnd</button>-->
        <form class="container" action="#" method="post" accept-charset="utf-8">
            <div v-if="showButtonScanner">
                <input v-if="captureObject" @click="beginScanning" id="scan" ref="scan" type="button" class="button is-primary pt-main-button" value="Start scanning">
                <!--<input v-else id="selectorElement" type="file" accept="image/*" value="Select an image" class="is-primary pt-main-button" name="ir"> -->
                <div v-else id="selectorElement">
                    <upload-file @file-chosen="updateFile" :isDisabled=isButtonDisabled></upload-file>
    
                    <!--<div v-if="file.name!==undefined" class="message">
                    "file-chosen" event received: <strong>{{ file.name }}</strong>
                    </div>-->
                </div>
            </div>
            <div v-if="showScanner" id="videoCapture" ref="videoCapture"></div>
        </form>	
        <div class="loader spinner is-invisible" id="spinner" ref="spinner"></div>
    </div>
</template>

<script>
import CraftAr from '../utils/craftar.min'
import UploadFile from './UploadFile'

export default {
    data:()=>({
        finderResults: false,
        cloudRecognition: null,
        captureObject: null,
        showScanner: true,
        showButtonScanner: true,
        capturerInitialized: false,
        file: '',
        isButtonDisabled: false,
        isFirstTime: true
    }),
    components: {
        UploadFile
    },
    created(){
        this.cloudRecognition =  new craftar.CloudRecognition({
            token: '4d1c9e52d5e24197'
        })
    },
    mounted(){
        var scanButton = document.getElementById('scan')

        if ( craftar.supportsCapture() ){
            let that = this
            this.setupCapture((err, captureObject) => {
                console.log("mounted > this.setupCapture")
                if (err){
                    console.log("mounted > err", err)
                    // Capture setup failed (user rejected to open the camera)
                    // switch to selector mode
                    that.switchToSelector()
                } else {
                    this.captureObject = captureObject        
                }

            })

        } else if (craftar.supportsImageSelector()){
            // Capture not supported, switch to selector mode
            this.switchToSelector();
        } else {
            alert("This browser doesn't support HTML5 features needed for CraftAR HTML5 Library");
        }
    },
    methods:{
        updateFile(file) {
            this.file = file;
        },
        beginScanning(){
            this.showScanner = true
            this.showButtonScanner = false
            if (this.isFirstTime) {
                var captureDivElement = document.getElementById('videoCapture')
                captureDivElement.appendChild(this.captureObject.domElement)
                this.isFirstTime = false
            }
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
            console.log("switchToSelector")
            var spinnerElement = document.getElementById('spinner');

            var selector = new craftar.ImageSelector(selectorElement);
            selector.addListener('image', (craftarImage) => {
                spinnerElement.setAttribute("class", "loader spinner")
                this.cloudRecognition.search(craftarImage)
            });

            this.setSelectorResultsListener()
        },

        setupCapture(callback){
            console.log("setupCapture", callback)
            var capture = new craftar.Capture();
            console.log("setupCapture > 2")
            {{debugger}}
            capture.addListener('started', () => {
                console.log("setupCapture > 3")
                callback(null, capture )
            })

            capture.addListener('error', (error) => {
                console.log("setupCapture > 4")
                callback(error, capture)
            })
            console.log("setupCapture > 5")
            capture.start()
        },

        setSelectorResultsListener() {
            this.cloudRecognition.addListener('results', (error, results, xhr) => {
                var spinnerElement = document.getElementById('spinner')
                spinnerElement.setAttribute("class", "loader spinner is-invisible")

                if (results.results && results.results.length > 0) {
                    this.renderResults(results);
                } else {
                    alert("No results found, select an image that contains an object to scan.")
                    this.editIsDisabled(false)
                }
            })
        },
        editIsDisabled(value){
            this.isButtonDisabled = value
        },
        setCaptureResultsListener() {
            this.cloudRecognition.addListener('results', (err, results, xhr) => {

                if (results.results && results.results.length > 0) {
                    this.finderResults = true
                    this.renderResults(results)
                    this.cloudRecognition.stopFinder()
                    var spinnerElement = document.getElementById('spinner')
                    spinnerElement.setAttribute("class", "loader spinner is-invisible")
                }
            })

            this.cloudRecognition.addListener('finderFinished', () => {
                var spinnerElement = document.getElementById('spinner')
                spinnerElement.setAttribute("class", "loader spinner is-invisible")
                if (!this.finderResults) {
                    alert("No results found, point to an object.")
                    this.showButtonScanner = true
                    this.isButtonDisabled = false
                    //this.$forceUpdate()
                }
            })
        },
        renderResults(results){
            this.isButtonDisabled = true
            //It is called once an item is recognized
            // results sample: {"search_time":87,"results":[{"item":{"name":"PET_Fantal_500ml","url":"promos.q.orchextra.io/test-guillermo-q","custom":"https://crs-beta-catchoom.s3.amazonaws.com/collections/a2a7e94186ef44b3be3f7dbf0ce70b49/metadata/592bbeb24ab446de8b6d2da17a2230ff.json","content":null,"trackable":false,"uuid":"592bbeb24ab446de8b6d2da17a2230ff"},"image":{"thumb_120":"https://crs-beta-catchoom.s3.amazonaws.com/cache/collections/a2a7e94186ef44b3be3f7dbf0ce70b49/images/592bbeb24ab446de8b6d2da17a2230ff_85371219b9ef4bc88e446a1b18d47764_thumb_120.jpe","uuid":"85371219b9ef4bc88e446a1b18d47764"},"score":59}]}
            var resultItem = results.results[0];

            //this.$emit('input', results.results)
            this.$parent.recognizedItems = Array.from(results.results)
            this.showButtonScanner = false
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