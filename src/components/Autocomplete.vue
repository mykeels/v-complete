<template>
    <div style="position:relative" class="v-complete" :class="{ 'open': openSuggestion }">
        <input class="form-control" type="text" :value="value" :placeholder="placeholder" :required="required"
          @input="updateValue($event.target.value)"
          @focus="show"
          @keydown.enter = 'enter'
          @keydown.down = 'down'
          @keydown.up = 'up'
          @keydown.esc = 'close'
          @blur = 'close'
        >
        <ul class="dropdown-menu full-width">
            <li v-for="(suggestion, index) in matches" :key="index"
                v-bind:class="{'active': isActive(index)}"
                @click="suggestionClick(index)"
            >
              <a>
                {{ suggestion.name }} <small v-if="suggestion.description">{{ suggestion.description }}</small>
              </a>
            </li>
        </ul>
    </div>
</template>

<script>
import 'string-compare'

export default {

  props: {

    value: {
      type: String,
      required: true
    },

    suggestions: {
      type: Array,
      required: true
    },

    /**
     * limits the number of autocomplete values shown at a time
     */
    limit: Number,

    /**
     * between 0 and 1, filters the autocomplete values based on how well they match the input value
     */
    threshold: {
      type: Number,
      default: 0
    },

    placeholder: String,

    required: Boolean,

    caseSensitive: Boolean

  },

  data () {
    return {
      open: false,
      current: 0
    }
  },

  computed: {
    // Filtering the suggestion based on the input
    matches () {
      return this.suggestions
              .map(s => (Object.assign({}, s, { $matchScore: this.value ? String.compare(this.resolveCase(s.name), this.resolveCase(this.value)) : 1 })))
              .filter(s => s.$matchScore > this.threshold)
              .sort((a, b) => a.$matchScore < b.$matchScore ? 1 : -1).slice(0, this.limit || 5)
    },

    openSuggestion () {
      return this.selection !== '' &&
             this.matches.length !== 0 &&
             this.open === true
    }
  },

  methods: {

    updateValue (value) {
      if (this.open === false) {
        this.open = true
        this.current = 0
      }
      this.$emit('input', value)
    },

    resolveCase (text) {
      if (!this.caseSensitive) {
        return (text || '').toLowerCase()
      }
      return text
    },

    // When enter pressed on the input
    enter () {
      if (this.matches[this.current]) this.$emit('input', this.matches[this.current].name)
      this.open = false
    },

    close () {
      setTimeout(() => {
        this.open = false
      }, 200)
    },

    show () {
      this.open = true
    },

    // When up pressed while suggestions are open
    up () {
      if (this.current > 0) {
        this.current--
      }
    },

    // When up pressed while suggestions are open
    down () {
      this.open = true
      if (this.current < this.matches.length - 1) {
        this.current++
      }
    },

    // For highlighting element
    isActive (index) {
      return index === this.current
    },

    // When one of the suggestion is clicked
    suggestionClick (index) {
      if (this.matches[this.current]) this.$emit('input', this.matches[index].name)
      this.open = false
    }

  }

}
</script>
