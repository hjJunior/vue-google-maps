<template>
  <slot v-bind="{ ref: input, ...$attrs }">
    <input ref="input" v-bind="$attrs" v-on="$attrs" />
  </slot>
</template>

<script>
import { bindProps, getPropsValues } from '../utils/bindProps.js'
import downArrowSimulator from '../utils/simulateArrowDown.js'
import { mappedPropsToVueProps } from './build-component'

const mappedProps = {
  bounds: {
    type: Object,
  },
  componentRestrictions: {
    type: Object,
    // Do not bind -- must check for undefined
    // in the property
    noBind: true,
  },
  types: {
    type: Array,
    default: function () {
      return []
    },
  },
}

const props = {
  selectFirstOnEnter: {
    required: false,
    type: Boolean,
    default: false,
  },
  options: {
    type: Object,
  },
}

export default {
  expose: ['build'],
  mounted() {
    if (this.$slots.default || this.$scopedSlots.default) return
    this.build();
  },
  methods: {
    build(ref) {
      const selfRef = ref ?? this.$refs.input;
      this.$gmapApiPromiseLazy().then(() => {
        if (this.selectFirstOnEnter) {
          downArrowSimulator(selfRef)
        }

        if (typeof google.maps.places.Autocomplete !== 'function') {
          throw new Error(
            "google.maps.places.Autocomplete is undefined. Did you add 'places' to libraries when loading Google Maps?"
          )
        }

        /* eslint-disable no-unused-vars */
        const finalOptions = {
          ...getPropsValues(this, mappedProps),
          ...this.options,
        }

        this.$autocomplete = new google.maps.places.Autocomplete(selfRef, finalOptions)
        bindProps(this, this.$autocomplete, mappedProps)

        this.$watch('componentRestrictions', (v) => {
          if (v !== undefined) {
            this.$autocomplete.setComponentRestrictions(v)
          }
        })

        // Not using `bindEvents` because we also want
        // to return the result of `getPlace()`
        this.$autocomplete.addListener('place_changed', () => {
          this.$emit('place_changed', this.$autocomplete.getPlace())
        })
      })
    }
  },
  props: {
    ...mappedPropsToVueProps(mappedProps),
    ...props,
  },
}
</script>
