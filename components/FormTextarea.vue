<template>
  <div class="form-group">
    <label v-if="$slots.label" :for="id" class="form-control-label"
      ><slot name="label"></slot
    ></label>
    <textarea-autosize
      :id="id"
      class="form-control"
      :rows="rows"
      :value="value"
      @input="$emit('input', $event)"
    ></textarea-autosize>
    <div v-if="$slots.text" class="form-text"><slot name="text"></slot></div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'FormTextarea',
  props: {
    value: {
      type: String,
      default: () => '',
      required: false,
    },
    rows: {
      type: String,
      default: () => '5',
      required: false,
    },
  },
  data: () => ({
    id: '',
  }),
  beforeMount() {
    this.id = this.getUniqueId()
  },
  methods: {
    getUniqueId() {
      return String(Math.floor(Math.random() * 10e10))
    },
  },
})
</script>
