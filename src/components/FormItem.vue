<template>
    <div>
        <label v-if="label" :class="{ 'i-form-item-label-required': isRequired }">{{ label }}</label>
        <div>
            <slot></slot>
            <div v-if="validateState === 'error'" class="i-form-item-message">{{ validateMessage }}</div>
        </div>
    </div>
</template>

<script>
/* eslint-disable */
import Emitter from '../../mixins/emitter.js'
import AsyncValidator from 'async-validator'

export default {
    inject: ['form'],
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
    data () {
        return {
            isRequired: false,
            initialValue: '',
            validateState: '',
            validateMessage: ''
        }
    },
    computed: {
        fieldValue () {
            return this.form.model[this.prop]
        }
    },
    methods: {
        setRules () {
            this.$on('on-form-blur', this.onFieldBlur)
            this.$on('on-form-change', this.onFieldChange)
        },
        getRules () {
            let formRules = this.form.rules
            formRules = formRules ? formRules[this.prop] : []
            return [].concat(formRules || [])
        },
        getFilteredRule (trigger) {
            const rules = this.getRules()
            return rules.filter(rule => !rule.trigger || rule.trigger.indexOf(trigger) !== -1)
        },
        resetField () {
            this.validateState = '';
            this.validateMessage = '';
            this.form.model[this.prop] = this.initialValue;
        },
        validate (trigger, callback = function () {}) {
            let rules = this.getFilteredRule(trigger)
            if (!rules || rules.length === 0) {
                return true
            }
            this.validateState = 'validating'
            let descriptor = {}
            descriptor[this.prop] = rules
            const validator = new AsyncValidator(descriptor)
            let model = {}
            model[this.prop] = this.fieldValue
            validator.validate(model, { firstFields: true }, errors => {
                this.validateState = !errors ? "success" : "error"
                this.validateMessage = errors ? errors[0].message : ''
            })
            callback(this.validateMessage)
        },
        onFieldBlur () {
            this.validate('blur')
        },
        onFieldChange () {
            this.validate('change')
        }
    },
    // 组件渲染时， 将实例缓存在Form中
    mounted () {
        if (this.prop) {
            this.dispatch('iForm', 'on-form-item-add', this)
            this.initialValue = this.fieldValue
            this.setRules()
        }
    },
    // 组件销毁前，将实例从Form的缓存中移除
    beforeDestory () {
        this.dispatch('iForm', 'on-form-item-remove', this)
    }
}
</script>

<style>
    .i-form-item-label-required:before {
        content: '*';
        color: red;
    }
    .i-form-item-message {
        color: red;
    }
</style>

