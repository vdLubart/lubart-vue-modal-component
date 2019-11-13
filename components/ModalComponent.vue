<template>
    <div class="modal-component" visible v-show="visibility" v-on:keyup.esc="closeModal">
        <div class="modal-component__background" @click="closeModal"></div>
        <div class="modal-component__card">
            <header class="modal-component__card__header">
                <p class="modal-component__card__header__title">{{ caption }}</p>
                <button class="modal-component__card__header__close" aria-label="close"></button>
            </header>
            <alert-component v-if="Object.keys(alertNotes).length" :notes="alertNotes" :status="alertType" :renderHtml="alertRenderHtml"></alert-component>
            <section class="modal-component__card__body" v-if="content" v-html="content"></section>
            <section class="modal-component__card__body" v-else>
                <slot></slot>
            </section>
            <footer class="modal-component__card__footer">
                <input-button v-show="cancelButtonVisibility" @click="cancelAction" :disabled="buttonsDisabled == 1" :label="cancelButton"></input-button>
                <input-button :disabled="buttonsDisabled == 1" :label="actionButtonText" @click="confirmModal"></input-button>
            </footer>
        </div>
    </div>
</template>

<script>
    import { AlertComponent } from 'lubart-alert-component';
    import { InputButton } from 'lubart-input-component';

    export default {
        name: "ModalComponent",

        components: { AlertComponent, InputButton },

        props: {
            title: {type: String},                                      // modal caption
            action: {type: String},                                     // method name in the root Vue instance. Option is
                                                                        // needed to customize modal confirmation event
            visible: {type: Boolean, default: false},                   // modal visibility
            actionButton: {type: String, default: "Save changes"},      // text for action button
            cancelButton: {type: String, default: "Cancel"},            // text for cancel button
            removeCancelButton: {type: Boolean, default: false},        // hide cancel button
            alert: {type: Object, default: () => {return {};} },
            alertStatus: {type: String, default: "success"},
            alertHtml: {type: Boolean, default: false}
        },

        data(){
            return {
                content: "",
                proxyData: {},                                          // set of data needed to transfer data back to the action method
                caption: this.title,
                submitAction: this.action,
                visibility: this.visible,
                actionButtonText: this.actionButton,
                cancelButtonText: this.cancelButton,
                cancelButtonVisibility: !this.removeCancelButton,
                buttonsDisabled: false,
                alertNotes: this.alert,
                alertRenderHtml: this.alertHtml,
                alertType: this.alertStatus
            }
        },

        watch: {
            title(val){
                this.caption = val;
            },

            action(val){
                this.submitAction = val;
            },

            visible(val){
                this.visibility = val;
            },

            actionButton(val){
                this.actionButtonText = val;
            },

            cancelButton(val){
                this.cancelButtonText = val;
            },

            removeCancelButton(val){
                this.cancelButtonVisibility = !val;
            },

            visibility(val){
                this.buttonsDisabled = !val;
            },

            alert(val){
                this.alertNotes = val;
            },

            alertStatus(val){
                this.alertType = val;
            }
        },

        methods: {
            /**
             * Event on click to modal confirm button
             */
            confirmModal(){
                this.buttonsDisabled = true;
                // run custom action from the $parent or any else component or close modal
                if(this.submitAction){
                    if(this.submitAction.includes('.')){
                        let vueComponent = this;

                        this.submitAction.split('.').forEach( (val) => {
                            vueComponent = vueComponent[val];
                        });

                        vueComponent(this.proxyData);
                    }
                    else {
                        this.$parent[this.submitAction](this.proxyData);
                    }
                }
                else{
                    this.closeModal();
                }
            },

            closeModal(){
                this.visibility = false;
                if(this.$parent.modalClosed) {
                    this.$parent.modalClosed();
                }
            },

            cancelAction(){
                if(this.$parent.cancelAction) {
                    this.$parent.cancelAction();
                }
                else{
                    this.closeModal();
                }
            }
        },

        mounted(){
            window.addEventListener('keyup', (e) => {
                if (e.key == 'Escape') {
                    this.closeModal();
                }
            });
        }
    }
</script>

<style scoped>

    .modal-component{
        display: flex;
        align-items: center;
        flex-direction: column;
        justify-content: center;
        overflow: hidden;
        position: fixed;
        z-index: 100;
        bottom: 0;
        left: 0;
        right: 0;
        top: 0;
    }

    .modal-component__background{
        background-color: rgba(10,10,10,0.5);
        bottom: 0;
        left: 0;
        position: absolute;
        right: 0;
        top: 0;
    }

    .modal-component__card{
        display: flex;
        flex-direction: column;
        max-height: calc(100vh - 40px);
        overflow: hidden;
        z-index: 100;
        min-width: 33%;
    }

    .modal-component__card__header{
        font-weight: bold;
    }

    .modal-component__card__header,
    .modal-component__card__footer{
        border-bottom: 1px solid #dbdbdb;
    }

    .modal-component__card__header,
    .modal-component__card__footer{
        align-items: center;
        background-color: #f5f5f5;
        display: flex;
        flex-shrink: 0;
        justify-content: flex-start;
        padding: 20px;
        position: relative;
    }

    .modal-component__card__footer{
        justify-content: flex-end;
    }

    .modal-component__card__body{
        background-color: #fff;
        flex-grow: 1;
        flex-shrink: 1;
        overflow: auto;
        padding: 20px;
    }

</style>