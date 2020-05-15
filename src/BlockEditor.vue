<template>
    <div class="form-row" :class="{'has-errors': error}">
        <label v-if="label">{{label}}</label>
        <iframe width="100%" class="frame" frameborder="0"></iframe>

        <div v-if="frameApp">
            <div v-for="(item, item_id) in frameApp.$refs.vmain.localConfig" :key="item_id">
                <div :key="uniqCounter">
                <modal :name="'modal-block-edit-'+item_id" width="600px" height="auto" :scrollable="true">
                    <div class="editPopup adminframe">
                        <vue-field v-for="field in frameApp.$refs.vmain.localConfig[item_id].item"
                                   :field="field"
                                   :key="field.name"
                                   @change="onChange"
                        ></vue-field>

                        <a href="#" class="button" @click="close($event, item_id)">Сохранить</a>
                    </div>
                </modal>
                </div>
            </div>
        </div>

        <input :type="'hidden'"
               :name="name"
               :value="value"
               :title="label"
               :placeholder="fieldParams.placeholder"
               :disabled="fieldParams.disabled"
               :maxlength="fieldParams.maxlength"
        />
        <span v-if="error" class="error">{{error}}</span>
    </div>
</template>

<script>
    import VueMain from "./VueMain";

    export default {
        name: "BlockEditor",
        props: {
            label: String,
            name: {type: String, required: true},
            value: [String, Number],
            error: String,
            fieldParams: {
                type: [Object, Array],
                default: function(){
                    return {
                        blocksListUrl: '/api/blocks',
                        blockFrontendUrl: '/api/block/frontned',
                        blockBackendUrl: '/api/block/backend',
                        css: [],
                        placeholder: '',
                        disabled: false,
                        maxlength: '',
                        rows: 15,
                    }
                }
            }
        },
        data() {
            return {
                realVal: {},
                frameApp: null,
                editorBody: null,
                editorHead: null,
                uniqCounter: 1,
                params: {},
                blocks: {
                    'header': {
                        frontend: '',
                        admin: ''
                    }
                }
            }
        },
        methods: {
            open(itemId) {
                this.$modal.show('modal-block-edit-'+itemId);
            },
            programCloseAll() {
                this.uniqCounter++;
            },
            onChange(value) {
                this.frameApp.$refs.vmain.$emit('change', this.frameApp.$refs.vmain.localConfig);
                this.frameApp.$refs.vmain.$forceUpdate();
            },
            close(e, itemId) {
                e.preventDefault();
                this.frameApp.$refs.vmain.isEdit[itemId] = false;
                this.frameApp.$refs.vmain.$forceUpdate();
                this.frameApp.$refs.vmain.$emit('close', this.frameApp.$refs.vmain);
            },
            updateFrameHeight() {
                setTimeout(() => {
                    let $frame = $('.frame');
                    let body = this.editorBody[0];
                    $frame.height($(this.editorBody).height() + 50 + 'px');
                }, 500);
            }
        },
        mounted() {
            this.realVal = JSON.parse(this.value);
            var vm = this;
            var styles = require('./style.css');
            this.params = this.fieldParams;
            let $frame = $('.frame');
            var idocument = $frame[0].contentDocument;
            idocument.open();
            idocument.write("<!DOCTYPE html>");
            idocument.write("<html>");
            idocument.write("<head></head>");
            idocument.write("<body></body>");
            idocument.write("</html>");
            idocument.close();
            this.editorBody = $frame.contents().find('body');
            this.editorHead = $frame.contents().find("head");
            this.fieldParams.css.forEach((item) => {
                this.editorHead.append('<link rel="stylesheet" href="'+item+'" />');
            });
            // this.editorHead.append('<style>.editPopup {display: block; background: #fff;position:fixed;top: 0;left: 0; width: 100%; height: 100%;}</style>');
            // this.editorBody.append('<h1>Блоковый редактор</h1>');

            let container = document.createElement('div');
            container.id = 'container';
            this.editorBody.append(container);

            vm.updateFrameHeight();

            this.frameApp = new Vue({
                el: container,
                data() {
                    return {
                        params: vm.fieldParams,
                        editConfig: vm.realVal,
                        blocks: vm.realVal.blocks,
                    }
                },
                components: {
                    VueMain,
                },
                methods: {
                    open(itemId) {
                        vm.open(itemId);
                    },
                    changeValue(value) {
                        let realVal = JSON.stringify(value.localConfig);
                        vm.$emit('input', realVal);
                        vm.updateFrameHeight();
                        vm.programCloseAll();
                    }
                },
                template: `<vue-main ref="vmain" @open="open" @close="changeValue" :items="blocks" :edit-config="editConfig" :params="params"></vue-main>`
            });

            this.frameApp.$on('update', () => {
                console.log('fream changed');
            });
        }
    }
</script>

<style scoped>

</style>