<template>
    <div>
        <li>
            <div id="thumbContainer">
                <a @click="sendEventParticipate" style="cursor: pointer;">
                    <img :src="thumbnailUrl">
                    <span>{{itemName}}</span> <span>[{{itemScore}}]</span>
                    <span>
                        <!--<a :href="itemUrl" target="_blank">Visit URL</a>-->
                    </span>
                </a>
            </div>
        </li>
    </div>
</template>

<script>
export default {
    props: ['thumbnailUrl','itemName','itemUrl','itemScore'],
    methods: {
            sendEventParticipate: function () {
                    //window.top.postMessage({type: this.messageTypes.ORCHEXTRA_PROMOTOOL_PARTICIPATE},'*')
                    this.$parent.showList = false
                    this.sendEventEnd()
            },
            sendEventEnd: function () {
                    //this.$parent.customEnd()
                    window.top.postMessage({type: this.messageTypes.ORCHEXTRA_PROMOTOOL_END},'*')
            }

    },
    data:()=>({
        greet:'hello',
        messageTypes: {
            ORCHEXTRA_PROMOTOOL_START: 'orchextraPromotoolStart',
            ORCHEXTRA_PROMOTOOL_PARTICIPATE: 'orchextraPromotoolParticipate',
            ORCHEXTRA_PROMOTOOL_END: 'orchextraPromotoolEnd'
        }
    }),
    created(){
        console.log(this.greet)
    }
}
</script>

<style lang="scss">

</style>