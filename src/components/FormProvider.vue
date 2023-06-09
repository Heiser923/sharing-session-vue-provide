<template>
  <form class="form-provider"
        @submit="submitEventHandler">
    <slot />
  </form>
</template>

<script>

export default {
  name: 'FormProvider',
  data() {
    return {
      fields: []
    }
  },
  provide() {
    return {
      // Arrow function because we need to access the this instance
      register: (component) => {
        if (this.fields.some(field => field.id === component.id)) {
          console.warn('warning the same id');
        } else {
          this.fields.push(
            component
          )
        }
      },
      unregister: (component) => {
        this.fields = this.fields.filter(field => field.id !== component.id)
      }
    }
  },
  methods: {
    async submitEventHandler(event) {
      event.preventDefault()

      const errors = []
      const validatorPromises = []

      console.log(this.fields)

      this.fields.forEach(
        field => {
          validatorPromises.push( new Promise(
            async () => {
              if ( await field.validatorRunner() ) {
                errors.push('error')
              }
            }
          )
          )
        }
      )

      await Promise.all(validatorPromises )

      if (errors.length) {
        return
      }

      this.$emit('submitted', event)
    }
  }
}
</script>

<style scoped>

  .form-provider {
    max-width: 80%;
  }

</style>