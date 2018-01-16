<template>
    <div class="hero is-fullheight">
        <div v-if="error"><h2 style="color: red;">Oops! Some error ocurred :(</h2></div>
        <div v-else>
            <div>
                <div v-if="recognizedItems">
                    <div v-if="showList" class="text-content container">
                        <h2 class="subtitle is-5">Is this your product?</h2>
                        <ul class="ir" ref="resultList" v-for="result in recognizedItems" :key='result.item.name'>
                            <recognized-item 
                                :thumbnail-url="result.image.thumb_120" 
                                :item-name="result.item.name" 
                                :item-url="result.item.url"
                                :item-score="result.score"></recognized-item>
                        </ul>
                    </div>
                </div>
                <recognizer v-else :buttonScanText='buttonText'></recognizer>
            </div>
        </div>
    </div>
</template>

<script>
import RecognizedItem from './RecognizedItem'
import Recognizer from './Recognizer'

    var orchextraPromoTool =  new Object({
        promoId: '1',
        language: 'en'
    })

export default {
    data:()=>({
        error: null,
        promoConfiguration: null,
        language: 'en',
        promoId: null,
        recognizedItems: null,
        messageTypes: {
            ORCHEXTRA_PROMOTOOL_START: 'orchextraPromotoolStart',
            ORCHEXTRA_PROMOTOOL_PARTICIPATE: 'orchextraPromotoolParticipate',
            ORCHEXTRA_PROMOTOOL_END: 'orchextraPromotoolEnd'
        },
        showList: true
    }),
    created(){
        this.useCases().sendEventStart(this)
        this.useCases().initPromotionConfiguration(this)
    },
    computed: {
        buttonText(){
            console.log("LANG", this.language)
            return this.promoConfiguration && this.promoConfiguration.data.template.iframeContent.button[this.language]
        }
    },
    components: {
        RecognizedItem,
        Recognizer
    },
    methods:{
        customStart(){
            window.top.postMessage({type: this.messageTypes.ORCHEXTRA_PROMOTOOL_START},'*')
        },
        customParticipate(){
            window.top.postMessage({type: this.messageTypes.ORCHEXTRA_PROMOTOOL_PARTICIPATE},'*')
        },
        customEnd(){
            window.top.postMessage({type: this.messageTypes.ORCHEXTRA_PROMOTOOL_END},'*')
        },
        makeRequest(method, url) {
            return new Promise(function (resolve, reject) {
                var xhr = new XMLHttpRequest();
                xhr.open(method, url);
                xhr.onload = function () {
                    if (this.status >= 200 && this.status < 300) {
                        resolve(xhr.response);
                    } else {
                        reject({
                            status: this.status,
                            statusText: xhr.statusText
                        });
                    }
                };
                xhr.onerror = function () {
                    reject({
                        status: this.status,
                        statusText: xhr.statusText
                    });
                };
                xhr.send();
            });
        },
        includeCSS(promoId, apiUrl){
            var cssId = 'orchextraCss';  // you could encode the css path itself to generate id..
            if (!document.getElementById(cssId))
            {
                var head  = document.getElementsByTagName('head')[0];
                var link  = document.createElement('link');
                link.id   = cssId;
                link.rel  = 'stylesheet';
                link.type = 'text/css';
                link.href = 'https://front-'+apiUrl+'/_template/'+promoId;
                link.media = 'all';
                head.appendChild(link);
            }
        },
        cleanUrl(url){
            console.log(url)
            if (url) {
                console.log(url.replace(/(^\w+:|^)\/\//, ''))
                return url.replace(/(^\w+:|^)\/\//, '')
            }
            return null
        },
        useCases(){
            return {
                initPromotionConfiguration: (context) => {
                    this.messageTypes = {}
                    var queryParams = window.location.search.substr(1).split('&').reduce(function (q, query) {
                            var chunks = query.split('=')
                            var key = chunks[0]
                            var value = chunks[1]
                            return (q[key] = value, q)
                    }, {})
                    context.promoId = queryParams['promoId']
                    if (!context.promoId){
                        let errorText = 'No promoId specified'
                        {{debugger}}
                        context.error = true
                        throw new ReferenceError(errorText)
                    }
                    context.language = queryParams['lang'] || 'en'
                    context.apiUrl = this.cleanUrl(queryParams['apiUrl']) || 'pt.orchextra.io'
                    //this.promoId = new URLSearchParams(window.location.search).get('promoId')
                    console.log('promoId', context.promoId)
                    //this.lang = new URLSearchParams(window.location.search).get('lang')
                    this.makeRequest('GET','https://'+context.apiUrl+'/configuration/'+context.promoId).then((response) => {
                        context.promoConfiguration = JSON.parse(response)
                        this.includeCSS(context.promoId, context.apiUrl)
                    }).catch((error) => {
                            throw new Error(error.statusText)
                        })
                    },
                sendEventStart: () => {
                    this.customStart()
                },
                participate: () => {
                    this.customParticipate()
                },
                sendEventEnd: () => {
                    this.customEnd()
                },
            }
        },
    }
}
</script>

<style lang="scss">
@import '~bulma';
</style>