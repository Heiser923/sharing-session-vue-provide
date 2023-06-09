<template>

  <input ref="input" v-model="value" />
  <span style="color: red;"> {{ errorMessage }} </span>

</template>

<script>

function generateRandomId(length=40) {
    let result = '';
    let characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    let charactersLength = characters.length;

    for (let i = 0; i < length; i++) {
        result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }

    return result;
}

export default {
  name: 'FormInput',
  inject: ['register', 'unregister'],
  data () {
    return {
			errorBucket: [],
			isValid: true,
			id: generateRandomId(),
    }
  },
  props: {
    rules: {
      type: Array,
      default: []
    },
    required: {
      type: Boolean,
      default: false,
    },
    modelValue: {
      type: String,
      default: ''
    }
  },
	beforeMount () {
		this.register(this)
	},
	beforeUnmount () {
		this.unregister(this)
	},
  methods: {
    async validatorRunner() {
			this.errorBucket = []
			await this.$nextTick()
			const inputElement = this.$refs.input
			inputElement.setCustomValidity('')

			this.validators.forEach(
				rule => {
					const validatedResult = rule(this.value)
					if (typeof validatedResult === 'string') {
						// HTML custom validity to trigger form to focus on it when submit.
						inputElement.setCustomValidity(' ')
						this.errorBucket.push(validatedResult)
					}
				}
			)
			return [ ...this.errorBucket ]
		}
  },
  computed: {
		validators() {
			const requiredRule = (value) => (!this.required || !!value) || 'This field is required'
			return [...this.rules, requiredRule]
		},
    errorMessage() {
			if (this.errorBucket.length) return this.errorBucket[0];
			return;
		},
		value: {
			get() {
				return this.modelValue;
			},
			set(value) {
				console.log( this.value, 'x')
				this.$emit('update:modelValue', value);
				this.validatorRunner()
			},
		},
  }
}
</script>