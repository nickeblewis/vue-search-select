<template>
  <div
    class="ui fluid search selection dropdown"
    :class="{ 'active visible':showMenu, 'error': isError, 'disabled': isDisabled }"
    @click="openOptions"
    @focus="openOptions"
  >
    <i class="dropdown icon"></i>
    <input
      class="search"
      autocomplete="off"
      :tabindex="isDisabled ? -1 : 0"
      :id="id"
      :name="name"
      :value="searchText"
      @input="searchText = $event.target.value"
      ref="input"
      @focus.prevent="openOptions"
      @keyup.esc="closeOptions"
      @blur="blurInput"
      @keydown.up="prevItem"
      @keydown.down="nextItem"
      @keydown.enter.prevent=""
      @keyup.enter.prevent="enterItem"
      @keydown.delete="deleteTextOrItem"
    />
    <div
      class="text"
      :class="textClass" :data-vss-custom-attr="searchTextCustomAttr"
    >{{inputText}}
    </div>
    <div
      class="menu"
      ref="menu"
      @mousedown.prevent
      :class="menuClass"
      :style="menuStyle"
      tabindex="-1"
    >
      <div
        v-for="(option, idx) in filteredOptions"
        :key="idx"
        class="item"
        :class="{ 'selected': option.selected || pointer === idx }"
        :data-vss-custom-attr="customAttrs[idx] ? customAttrs[idx] : ''"
        @click.stop="selectItem(option)"
        @mousedown="mousedownItem"
        @mouseenter="pointerSet(idx)"
      >
        {{option.text}}
      </div>
    </div>
  </div>
</template>

<script>
import common from './common'
import { baseMixin, commonMixin, optionAwareMixin } from './mixins'

export default {
  name: 'BasicSelect',
  mixins: [baseMixin, commonMixin, optionAwareMixin],
  props: {
    selectedOption: {
      type: Object,
      default: () => { return { value: '', text: '' } }
    }
  },
  data () {
    return {
      showMenu: false,
      searchText: '',
      mousedownState: false, // mousedown on option menu
      pointer: -1
    }
  },
  watch: {
    selectedOption (newValue) {
      if (newValue && newValue.value) {
        this.pointer = this.filteredOptions.findIndex(option => {
          return option.value === newValue.value
        })
      } else {
        this.pointer = -1
      }
    }
  },
  computed: {
    searchTextCustomAttr () {
      if (this.selectedOption && this.selectedOption.value) {
        return this.customAttr(this.selectedOption)
      }
      return ''
    },
    inputText () {
      if (this.searchText) {
        return ''
      } else {
        let text = this.placeholder
        if (this.selectedOption.text) {
          text = this.selectedOption.text
        }
        return text
      }
    },
    customAttrs () {
      try {
        if (Array.isArray(this.options)) {
          return this.options.map(o => this.customAttr(o))
        }
      } catch (e) {
        // if there is an error, just return an empty array
      }
      return []
    },
    textClass () {
      if (!this.selectedOption.text && this.placeholder) {
        return 'default'
      } else {
        return ''
      }
    },
    menuClass () {
      return {
        visible: this.showMenu,
        hidden: !this.showMenu
      }
    },
    menuStyle () {
      return {
        display: this.showMenu ? 'block' : 'none'
      }
    },
    filteredOptions () {
      if (this.searchText) {
        return this.options.filter((option) => {
          try {
            return this.filterPredicate(option.text, this.searchText)
          } catch (e) {
            return true
          }
        })
      } else {
        return this.options
      }
    }
  },
  methods: {
    deleteTextOrItem () {
      if (!this.searchText && this.selectedOption) {
        this.selectItem({})
        this.openOptions()
      }
    },
    openOptions () {
      common.openOptions(this)
    },
    blurInput () {
      common.blurInput(this)
    },
    closeOptions () {
      common.closeOptions(this)
    },
    prevItem () {
      common.prevItem(this)
    },
    nextItem () {
      common.nextItem(this)
    },
    enterItem () {
      common.enterItem(this)
    },
    pointerSet (index) {
      common.pointerSet(this, index)
    },
    pointerAdjust () {
      common.pointerAdjust(this)
    },
    mousedownItem () {
      common.mousedownItem(this)
    },
    selectItem (option) {
      this.searchText = '' // reset text when select item
      this.closeOptions()
      this.$emit('select', option)
      if (option.value === option.text) {
        this.searchText = option.value
      }
    }
  }
}
</script>
