<template>
    <div class="hero is-fullheight">
        <div>
            <div v-if="recognizedItems">
                <div v-if="showList" class="text-content container">
                    <h2 class="subtitle is-5">Is this your product?</h2>
                    <ul class="ir" ref="resultList" v-for="result in recognizedItems" :key='result.item.name'>
                        <recognized-item :thumbnail-url="result.image.thumb_120" :item-name="result.item.name" :item-url="result.item.url"></recognized-item>
                    </ul>
                </div>
            </div>
            <recognizer v-else></recognizer>
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
        greet:'hello',
        recognizedItems: null,
        messageTypes: {
            ORCHEXTRA_PROMOTOOL_START: 'orchextraPromotoolStart',
            ORCHEXTRA_PROMOTOOL_PARTICIPATE: 'orchextraPromotoolParticipate',
            ORCHEXTRA_PROMOTOOL_END: 'orchextraPromotoolEnd'
        },
        showList: true
    }),
    created(){
        console.log(this.greet)
        this.useCases().sendEventStart()
        this.useCases().initPromotionConfiguration()
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
        includeCSS(promoId){
            var cssId = 'orchextraCss';  // you could encode the css path itself to generate id..
            if (!document.getElementById(cssId))
            {
                var head  = document.getElementsByTagName('head')[0];
                var link  = document.createElement('link');
                link.id   = cssId;
                link.rel  = 'stylesheet';
                link.type = 'text/css';
                link.href = 'https://front-pt.orchextra.io/_template/'+promoId;
                link.media = 'all';
                head.appendChild(link);
            }
        },
        useCases(){
            return {
                initPromotionConfiguration: () => {
                    this.messageTypes = {};
                    var queryParams = window.location.search.substr(1).split('&').reduce(function (q, query) {
                            var chunks = query.split('=');
                            var key = chunks[0];
                            var value = chunks[1];
                            return (q[key] = value, q);
                    }, {});
                    this.promoId = queryParams['promoId'];
                    this.lang = queryParams['lang'] || 'en';
                    //this.promoId = new URLSearchParams(window.location.search).get('promoId')
                    console.log('promoId', this.promoId)
                    //this.lang = new URLSearchParams(window.location.search).get('lang')
                    this.makeRequest('GET','https://pt.orchextra.io/configuration/'+this.promoId).then((response) => {
                        this.promoConfiguration = response
                        this.includeCSS(this.promoId)
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