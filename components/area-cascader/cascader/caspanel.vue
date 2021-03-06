<template>
    <span>
        <ul class="cascader-menu-list" ref="list">
            <li v-for="(item, index) in data"
                :key="getUniqueKey(index)" 
                :class="{
                    'cascader-menu-option': true,
                    'cascader-menu-extensible': item['children'],
                    'selected': cascader.useTmp ? cascader.tmpVals.includes(item.value) : cascader.vals.includes(item.value)
                }" 
                @click.stop="handleClickItem(item)">
                {{item.label}}
            </li>
        </ul>
        <caspanel v-if="sublist && sublist.length" :data="sublist"></caspanel>
    </span>
</template>

<script>
    import { isArray, scrollIntoView, setCSS, getCSS } from '@src/utils';
    import Emitter from '../mixins/emitter';

    let key = 0;

    export default {
        name: 'Caspanel',
        inject: ['cascader'],
        mixins: [Emitter],

        props: {
            data: {
                type: Array,
                default: () => []
            }
        },

        data () {
            return {
                sublist: [],
                scrollbar: null,
                // 记录上一次点击的 item
                oldItem: {},
                val: '',
                list: null
            };
        },

        watch: {
            data () {
                this.sublist = [];
            }
        },

        methods: {
            getUniqueKey () {
                return key++;
            },

            getBaseItem (item) {
                const backItem = Object.assign({}, item);
                if (backItem.children) {
                    delete backItem.children;
                }
                return backItem;
            },

            handleClickItem (item) {
                this.cascader.eventBus.$emit('set-use-tmp', true);
                this.triggerItem(item, 'click');
            },

            triggerItem (item, from) {
                const base = this.getBaseItem(item);
                this.cascader.eventBus.$emit('item-click', base, this.oldItem);
                this.oldItem = Object.assign({}, base);
                this.tmp = Object.assign({}, base);

                if (item.children && isArray(item.children) && item.children.length > 0) {
                    this.sublist = [].concat(item.children);
                } else {
                    this.sublist = [];
                    this.cascader.eventBus.$emit('hide-menu-wrap', from);
                }
            },

            initCaspanel (params) {
                const val = params.value;
                const value = [...val];

                for (let i = 0; i < value.length; i++) {
                    for (let j = 0; j < this.data.length; j++) {
                        if (value[i] === this.data[j].value) {
                            this.triggerItem(this.data[j], params.from);
                            this.val = value[i];
                            value.splice(0, 1);
                            this.$nextTick(() => {
                                this.broadcast('Caspanel', 'update-selected', {
                                    value: value,
                                    from: params.from
                                });
                            });
                            break;
                        }
                    }
                }
            },

            scrollToSelectedOption () {
                if (!this.list) {
                    this.list = this.$refs.list;
                }
                const seletedOption = this.data.filter(item => item.value === this.val);
                if (seletedOption.length && this.list) {
                    const target = this.list.querySelector('.selected');
                    scrollIntoView(this.list, target);
                }
            }
        },

        mounted () {
            // 保存 ul
            this.list = this.$refs.list;
            // 根据 value 初始化选择数据
            this.$on('update-selected', this.initCaspanel);
            this.cascader.eventBus.$on('set-scroll-top', this.scrollToSelectedOption);
        },

        beforeDestroy () {
            this.scrollbar && this.scrollbar.destroy();
        }
    };
</script>
