<template>
    <div>


        <draggable @change="onDrop" name="test" v-model="localConfig" class="_list">
        <div v-for="(item, item_id) in localConfig" :key="item_id">
            <div class="block-adminframe" v-if="item.block">
                <div v-html="renderMustache(item.block.frontend_template, localConfig[item_id].item, item_id)"></div>
                <div class="adminframe-editwrapper">
                    <div>
                        <a class="adminframe-updatelink" @click="update($event, item_id)" href="#">Обновить</a>
                    </div>
                    <div>
                        <a class="adminframe-editlink" @click="edit($event, item.block.admin_template, item_id)" href="#">Редактировать</a>
                    </div>
                    <div>
                        <a class="adminframe-deletelink" @click="drop($event, item_id)" href="#">Удалить</a>
                    </div>
                </div>

            </div>
        </div>
        </draggable>

        <hr style="margin-top: 40px">

        <select id="block_select+" @change="onSelectBlock">
            <option value="0">Выбор</option>
            <option :value="item.id" v-for="item in blocks">
                {{ item.name }}
            </option>
        </select>

        <div v-if="isDebug">
            <hr>
            Отладка:
            <hr>
            log: {{log}}
            <hr>
            isEdit: {{ isEdit }}
            <hr>
            localConfig: {{ localConfig }}
            <hr>
            params: {{ params }}
            <hr>
            blocks: {{ blocks }}
            <hr>
            items: {{items}}
        </div>
    </div>
</template>

<script>
    import Mustache from 'mustache';
    import draggable from 'vuedraggable';
    import {clone} from 'lodash';

    export default {
        name: "VueMain",
        props: ['params', 'editConfig', 'items'],
        components: {
            draggable
        },
        data() {
            return {
                order: [],
                isDebug: false,
                isEdit: [],
                counter: 1,
                localConfig: [],
                blocks: [],
                log: [],
            };
        },
        computed: {
            myList: {
                set(value) {
                    console.log(value);
                    this.localConfig = value;
                    this.$emit('close', this);
                },
                get() {
                    return this.localConfig;
                }
            }
        },
        mounted() {
            setTimeout( () => {
                this.loadBlockList();
            }, 500);
            this.order = clone(this.items);
            this.localConfig = clone(this.editConfig);
            this.$forceUpdate();
        },
        methods: {
            onDrop() {
                this.$emit('close', this);
            },
            onChange(value) {
                log.push('change');
                this.$emit('change', this.localConfig);
                this.$forceUpdate();
            },
            update(e, itemId) {
                e.preventDefault();
                Request.post(this.params.blockUrl, {
                    id: this.localConfig[itemId].block.id
                }, (response) => {
                    this.localConfig[itemId].block = response.block;
                    this.$forceUpdate();
                    this.$emit('close', this);
                });
            },
            close(e, itemId) {
                e.preventDefault();
                this.isEdit[itemId] = false;
                this.$forceUpdate();
                this.$emit('close', this);
            },
            drop(e, itemId) {
                e.preventDefault();
                if (!confirm('Вы уверены?')) {
                    return;
                }
                this.localConfig.splice(itemId, 1);
                this.$forceUpdate();
                this.$emit('close', this);
            },
            edit(e, adminConfig, itemId) {
                e.preventDefault();
                this.isEdit.splice(itemId, 1);
                this.isEdit[itemId] = 'ok';
                if (typeof (this.localConfig[itemId]['item']) === 'undefined') {
                    this.localConfig[itemId]['item'] = JSON.parse(adminConfig);
                }
                this.$emit('open', itemId);
                this.$forceUpdate();
                this.$parent.$forceUpdate();
            },
            increment() {
                this.counter++;
            },
            renderMustache(template, item, itemId) {
                if (!item) {
                    return '';
                }
                if (item) {
                    item.image = function () {
                        return function(text, render) {
                            let id = item[text].value;
                            if (!id) {
                                return '';
                            }
                            return item[text].fieldParams.data[id].url;
                        }
                    }
                }
                item.index = itemId;

                return Mustache.render(template, item);
            },
            onSelectBlock(e) {
                let val = e.target.value;
                if (val === 0) {
                    return;
                }
                Request.post(this.params.blockUrl, {
                    id: val
                }, (response) => {
                    if (!this.localConfig) {
                        this.localConfig = [];
                    }
                    let item = JSON.parse(response.block.admin_template);
                    this.localConfig.push({
                        block: response.block,
                        item: item,
                    });
                    this.$parent.$forceUpdate();
                });
            },
            loadBlockList() {
                var token = $('meta[name="csrf-token"]').attr('content');
                Request.post(this.params.blocksListUrl, {
                    data: {
                        _token: token
                    }
                }, (response) => {
                    this.blocks = response.blocks;
                });
            }
        },
    }
</script>

<style scoped>
</style>