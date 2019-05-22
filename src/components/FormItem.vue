<template>
    <div>
        <label v-if="label">{{ label }}</label>
        <div>
            <slot></slot>
        </div>
    </div>
</template>

<script>
import Emitter from '../../mixins/emitter.js'

export default {
    name:'iFormItem',
    mixins: [ Emitter ],
    props: {
        label: {
            type: String,
            default: ''
        },
        prop: {
            type: String
        }
    },
    // 组件渲染时， 将实例缓存在Form中
    mounted () {
        if (this.prop) {
            this.dispatch('iForm', 'on-form-item-add', this)
        }
    },
    // 组件销毁前，将实例从Form的缓存中移除
    beforeDestory () {
        this.dispatch('iForm', 'on-form-item-remove', this)
    }
}
</script>

