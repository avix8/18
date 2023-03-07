<template>
  <div style="width: 100%" :class="`align-${align}`">
    <v-autocomplete
      v-if="filtered.length"
      v-model="selected"
      v-click-outside="selectValue"
      :dense="dense"
      :hide-details="hideDetails"
      validate-on-blur
      hide-no-data
      :disabled="disabled"
      :outlined="outlined"
      :items="allItems"
      :label="label"
      :search-input.sync="searchValue"
      @keydown.tab="selectValue"
    ></v-autocomplete>
    <!-- @update:search-input="searchInput" -->
    <v-text-field
      v-else
      :dense="dense"
      :hide-details="hideDetails"
      :outlined="outlined"
      :clearable="clearable"
      :disabled="disabled"

      :label="label"
      :value="value"
      @input="input"
    ></v-text-field>
  </div>
</template>

<script>
export default {
  props: {
    value: { type: String, default: '' },
    align: { type: String, default: 'left' },
    label: { type: String, default: '' },
    clearable: { type: Boolean, default: true },
    outlined: { type: Boolean, default: false },
    dense: { type: Boolean, default: false },
    disabled: { type: Boolean, default: false },
    hideDetails: { type: Boolean, default: false },
    items: {
      type: Array,
      default: () => {
        return []
      },
    },
  },
  data: () => ({
    searchValue: '',
    selected: '',
    isSelecting: false,
  }),
  computed: {
    filtered() {
      return this.items.filter((x) => x)
    },
    allItems() {
      if (this.value.length === 0 || this.isSelecting) return this.filtered
      return [this.value].concat(this.filtered)
    },
  },
  watch: {
    searchValue(val) {
      this.isSelecting = true
      if (val !== null) this.$emit('input', val)
      else this.selectValue()
    },
  },
  methods: {
    setSelecting() {
      this.isSelecting = true
    },
    input(val) {
      this.$emit('input', val || '')
    },
    selectValue() {
      this.isSelecting = false
      this.selected = this.value
    },
  },
}
</script>

<style scoped>
.align-start >>> input, .align-left >>> input {
  text-align: left
}

.align-middle >>> input, .align-center >>> input {
  text-align: center
}

.align-end >>> input, .align-right >>> input {
  text-align: right
}

.v-input >>> input {
  min-width: 0px !important;
}
</style>
