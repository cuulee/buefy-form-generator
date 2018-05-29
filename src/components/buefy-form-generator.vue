<template>
	<div class="columns is-multiline">
		<template v-for="(input,key,index) in schema">
			<div class="column is-12" v-if="input.header">
				<div>
					<template v-if="index !== 0"><hr><br></template>
					<h1 class="title is-5">
						<b-icon style="transform:translateY(2px)" class="margin-right-xsmall" :icon="input.header.icon" />
						{{input.header.text}}
					</h1>
				</div>
			</div>

			<div :class="input.appearance.layout">
				<div>
					<b-field
					:class="(input.appearance.classField || '') + ' field-' + input.data.type"
					:label="labelGen(input.data.type, input.appearance.label, input.data.options || null)">
						<div>
							<template v-if="input.data.type === 'html'">
								<div v-html="input.data.html"></div>
							</template>

							<template v-else-if="input.data.type === 'link'">
								<a :href="input.data.linkTo" :target="input.data.targetTo">{{input.data.text}}</a>
							</template>

							<template v-else-if="inputTypes.indexOf(input.data.type) > -1">
								<b-input
								:name="key"
								:type="input.data.type === 'phone' ? 'text' : input.data.type"
								:class="input.appearance.classInput || ''"
								:icon="input.appearance.icon"
								:readonly="input.data.readonly || false"
								:disabled="input.data.disabled || false"
								:placeholder="input.data.placeholder || input.appearance.label || ''"
								v-model="input.data.value"
								v-validate="input.data.validate || ''"
								:data-vv-as="input.appearance.label"
								@input="emitter($event, key, 'changed')"
								/>
							</template>

							<template v-else-if="input.data.type === 'switch'">
								<b-switch
								:name="key"
								:type="input.data.type"
								:class="input.appearance.classInput || ''"
								:disabled="input.data.disabled || false"
								true-value="Yes"
								false-value="No"
								v-model="input.data.value"
								@input="emitter($event, key, 'changed')"
								>{{input.data.value}}</b-switch>
							</template>

							<template v-else-if="input.data.type === 'checkbox'">
								<b-checkbox
								:name="key"
								:type="input.data.type"
								:class="input.appearance.classInput || ''"
								:disabled="input.data.disabled || false"
								v-model="input.data.value"
								@input="emitter($event, key, 'changed')"
								><b>{{input.appearance.label}}</b></b-checkbox>
							</template>

							<template v-else-if="input.data.type === 'select'">
								<b-select
								:name="key" v-model="input.data.value"
								:class="input.appearance.classInput || ''"
								:icon="input.appearance.icon"
								:placeholder="input.data.placeholder || input.appearance.label || ''"
								:disabled="input.data.options ? input.data.options.length === 0 : true"
								:loading="input.data.options ? input.data.options.length === 0 : true"
								@input="emitter($event, key, 'changed')"
								>
									<option v-for="(option,index) in input.data.options" :value="option.value" :key="index">
										{{ option.text }}
									</option>
								</b-select>
							</template>

							<template v-else-if="input.data.type === 'date'">
								<b-datepicker
								:name="key"
								:class="input.appearance.classInput || ''"
								:icon="input.appearance.icon"
								:placeholder="input.data.placeholder || input.appearance.label || ''"
								:disabled="input.data.disabled || false"
								v-model="input.data.value"
								v-validate="input.data.validate || 'date_format:DD/MM/YYYY'"
								:data-vv-as="input.appearance.label"
								@input="emitter($event, key, 'changed')"
								/>
							</template>

							<template v-else-if="input.data.type === 'dropzone'">
								<dropzoneWrap
								:keyName="key"
								:class="input.appearance.classInput || ''"
								:value="input.data.value"
								:options="input.data.options"
								:button="input.data.button"
								@input="emitter($event, key, 'changed')"
								@cancel="$emit('canceled', {value: $event, name: key})"
								/>
							</template>

							<template v-else-if="input.data.type === 'richtext'">
								<quillEditor
								:ref="'quill-' + key"
								:name="key"
								:class="input.appearance.classInput || ''"
								:config="input.data.options"
								v-model="input.data.value"
								@input="emitter($event, key, 'changed')"
								/>
							</template>
						</div>
					</b-field>

					<button class="button is-danger"
					v-if="cancelable(input)"
					@click="schema[key].data.value = null; emitter($event, key, 'canceled')"
					>Cancel</button>

					<div class="v-error tc-red margin-top-xsmall">
						<vv-error :for="key" />
					</div>
				</div>
			</div>
		</template>
	</div>
</template>

<script>
import Vue from 'vue'

import quillEditor from './quillEditor.vue'
import dropzoneWrap from './dropzoneWrap.vue'

import Datepicker from 'buefy/src/components/datepicker/Datepicker.vue'
import Field from 'buefy/src/components/field/Field.vue'
import Select from 'buefy/src/components/select/Select.vue'
import Input from 'buefy/src/components/input/Input.vue'
import Checkbox from 'buefy/src/components/checkbox/Checkbox.vue'
import Switch from 'buefy/src/components/switch/Switch.vue'
import Message from 'buefy/src/components/message/Message.vue'
import Icon from 'buefy/src/components/icon/Icon.vue'
import Loading from 'buefy/src/components/loading/Loading.vue'

import VeeValidate, { Validator, ErrorComponent } from 'vee-validate'

import VueScrollTo from 'vue-scrollto'

Vue.component('quillEditor', quillEditor)
Vue.component('dropzoneWrap', dropzoneWrap)

Vue.component(Datepicker.name, Datepicker)
Vue.component(Field.name, Field)
Vue.component(Select.name, Select)
Vue.component(Input.name, Input)
Vue.component(Checkbox.name, Checkbox)
Vue.component(Switch.name, Switch)
Vue.component(Message.name, Message)
Vue.component(Icon.name, Icon)
Vue.component(Loading.name, Loading)

const vvConfig = { enableAutoClasses: true, locale: 'it', events: 'blur', errorBagName: 'vErrors' }
Vue.use(VeeValidate, vvConfig)
Vue.component('vv-error', ErrorComponent)

Vue.use(VueScrollTo)

var PhoneNumber = require('awesome-phonenumber')

export default {
	name: 'buefyFormGenerator',
	data () {
		return {
			canUpload: {},
			processing: true,
			original_schema: JSON.parse(JSON.stringify(this.schema)),
			inputTypes: ['text', 'phone', 'password', 'number', 'textarea']
		}
	},
	props: {
		schema: { default: {} }
	},
	computed: {
		schemaData () {
			let wholeSchema = this.schema
			let schemaKeys = Object.keys(wholeSchema)
			let schemaValues = Object.values(wholeSchema)
			let schemaData = {}

			for (let i = 0; i < schemaValues.length; i++) {
				let key = schemaKeys[i]
				let val = schemaValues[i].data

				if (!val.noSend) {
					let assign = val.value
					if (val.type === 'date') assign = require('moment').utc(assign).format('DD/MM/YYYY')
					if (val.type === 'switch') assign = (assign === 'Yes')
					schemaData[key] = assign
				}
			}

			return schemaData
		}
	},
	created () {
		this.$validator.extend('phone', {
			getMessage: field => this.$t('pages.account.settings.invalid_phone'),
			validate: value => new PhoneNumber(value, this.$store.state.locale).isValid()
		})
	},
	methods: {
		saveToOriginalSchema (schema) {
			// save only once
			if (!Object.keys(this.original_schema).length) this.original_schema = JSON.parse(JSON.stringify(schema))
		},
		cancelable (input) {
			if (input && input.data) {
				return (input.data.type !== 'html') && (input.data.type !== 'switch') && (input.data.type !== 'checkbox') && input.data.cancelable
			} else return false
		},
		labelGen (type, sourceLabel, options) {
			let label = null
			let fileTypes = null

			if (type !== 'checkbox') label = sourceLabel
			if (type === 'file') {
				if (options) fileTypes = options.acceptedFiles.toString().replace(/,/g, ', ')
				label = sourceLabel + ' (' + fileTypes + ')'
			}

			return label
		},
		validateSingle (e) {
			let valType = (typeof e)
			let inputName = valType === 'string' ? e : e.target.getAttribute('name')

			setTimeout(a => { this.$validator.validate(inputName, this.schema[inputName].data.value) }, 10)
		},
		validateAll () {
			this.$validator.validateAll().then(res => {
				this.$emit('validation', res)

				if (!res) {
					this.$nextTick(a => {
						let errors = document.querySelectorAll('.v-error')
						console.log([...errors].filter(v => { if (v.childNodes[0].innerHTML) return v }))
						VueScrollTo.scrollTo(errors[0], 500, { offset: -200 })
					})
				}
			}).catch(e => console.log(e))
		},
		emitter (e, key, eventToEmit) {
			if (this.schema[key].data.validate) if (this.original_schema[key].data.value || this.vErrors.has(key)) this.validateSingle(key)
			this.$emit(eventToEmit, {value: this.schemaData[key], value_native: e, name: key})
		}
	},
	watch: {
		'schema' (newVal, oldVal) { this.saveToOriginalSchema(newVal) }
	}
}
</script>