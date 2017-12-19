<template>
    <div>
        <!-- PRETTY FILE INPUT HIDDEN -->
        <div class="input-group">
            <span class="input-group-btn">
                <button ref="buttonFile" class="is-primary pt-main-button" type="button" @click="launchFilePicker" :disabled=isDisabled><span aria-hidden="true">CHOOSE A FILE</span></button>
            </span>
        </div>

        <!-- REAL FILE INPUT HIDDEN -->
        <input name="ir" type="file" style="display:none" ref="file" v-uploader />
  </div>
</template>

<script>
export default {
    name: 'upload-file',
    props: ['isDisabled'],
    data:()=>({
        file: ''
    }),
    directives: {
        uploader: {
            bind(el, binding, vnode) {
                el.addEventListener('change', e => {
                vnode.context.file = e.target.files[0];
                });
            }
        },
    },
    watch: {
        file(val) {
            this.$parent.isButtonDisabled = true // change to emit
            this.$refs.buttonFile.setAttribute("class", "button is-primary pt-main-button is-disabled")
            this.$emit('file-chosen', val)
        },
        isDisabled(val){
            if (val) {
                this.$refs.buttonFile.setAttribute("class", "button is-primary pt-main-button is-disabled")
            } else {
                this.$refs.buttonFile.setAttribute("class", "button is-primary pt-main-button")
            }
        }

    },
    methods: {
        launchFilePicker() {
            this.$refs.file.click();
        }
    }
}
</script>

<style lang="scss">
@import '~bulma';
</style>