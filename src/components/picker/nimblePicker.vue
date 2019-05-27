<template>
  <div class="emoji-mart" :style="customStyles">
    <div class="emoji-mart-bar emoji-mart-bar-anchors" v-if="showCategories">
      <anchors
        :i18n="mergedI18n"
        :color="color"
        :categories="categories"
        :active-category="activeCategory"
        @click="onAnchorClick"
      />
    </div>

    <slot
      name="searchTemplate"
      v-model="searchText"
      :i18n="mergedI18n"
    >
      <div class="emoji-mart-search">
        <input
          type="text"
          class="emoji-mart-search-input"
          v-model="searchText"
          :placeholder="mergedI18n.search"
          :autoFocus="autoFocus"
        />
      </div>
    </slot>

    <VirtualList
      v-if="items.length"
      class="scroller"
      :size="36"
      :remain="8"
      :start="startFromIdx"
      :item="CategoryItem"
      :itemcount="items.length"
      :itemprops="getItemprops"
      :onscroll="onScrollUpdate"
    />
    <div
      v-else
      class="emoji-mart-no-results"
    >
      {{ mergedI18n.notfound }}
    </div>

    <slot
      name="previewTemplate"
      :data="data"
      :title="title"
      :emoji="previewEmoji"
      :idle-emoji="idleEmoji"
      :show-skin-tones="showSkinTones"
      :emoji-props="emojiProps"
      :skin-props="skinProps"
      :on-skin-change="onSkinChange"
    >
      <div class="emoji-mart-bar emoji-mart-bar-preview" v-if="showPreview">
        <preview
          :data="data"
          :title="title"
          :emoji="previewEmoji"
          :idle-emoji="idleEmoji"
          :show-skin-tones="showSkinTones"
          :emoji-props="emojiProps"
          :skin-props="skinProps"
          :on-skin-change="onSkinChange"
        />
      </div>
    </slot>
  </div>
</template>

<script>
import '../../vendor/raf-polyfill'

import VirtualList from 'vue-virtual-scroll-list'
import chunk from 'lodash/chunk'
import findLast from 'lodash/findLast'
import find from 'lodash/find'

import store from '../../utils/store'
import frequently from '../../utils/frequently'
import { deepMerge, measureScrollbar } from '../../utils'
import { EmojiIndex } from '../../utils/emoji-data'
import { PickerProps } from '../../utils/shared-props'
import Anchors from '../anchors'
import Category from '../category'
import Preview from '../preview'
import CategoryItem from '../CategoryItem'

const I18N = {
  search: 'Search',
  notfound: 'No Emoji Found',
  categories: {
    search: 'Search Results',
    recent: 'Frequently Used',
    people: 'Smileys & People',
    nature: 'Animals & Nature',
    foods: 'Food & Drink',
    activity: 'Activity',
    places: 'Travel & Places',
    objects: 'Objects',
    symbols: 'Symbols',
    flags: 'Flags',
    custom: 'Custom',
  },
}

export default {
  components: {
    VirtualList,
    Anchors,
    Category,
    Preview,
  },

  props: {
    ...PickerProps,
    data: {
      type: EmojiIndex,
      required: true,
    },
  },

  data () {
    const categories = [
      ...this.data.categories().filter(category => (category.emojis.length > 0))
    ]

    return {
      activeSkin: this.skin || store.get('skin') || this.defaultSkin,
      activeCategory: categories[0],
      previewEmoji: undefined,
      startFromIdx: 0,
      searchText: '',
      categories: Object.freeze(categories),
      CategoryItem,
    }
  },

  computed: {
    items () {
      if (this.searchText) {
        const emojis = this.data.search(this.searchText, this.maxSearchResults)
        return chunk(emojis, 8)
      }

      return this.categories.reduce((out, category) => {
        if (!category.emojis.length) {
          return out
        }

        return out.concat({
          category: {
            id: category.id,
            name: category.name,
            first: category.first,
          },
        }, chunk(category.emojis, 8))
      }, [])
    },

    customStyles() {
      return {
        width: this.calculateWidth + 'px',
        ...this.pickerStyles,
      }
    },

    emojiProps() {
      return {
        native: this.native,
        skin: this.activeSkin,
        set: this.set,
        emojiTooltip: this.emojiTooltip,
        emojiSize: this.emojiSize
      }
    },

    skinProps() {
      return {
        skin: this.activeSkin,
      }
    },

    calculateWidth() {
      return this.perLine * (this.emojiSize + 12) + 12 + 2 + measureScrollbar()
    },

    mergedI18n() {
      return Object.freeze(deepMerge(I18N, this.i18n))
    },

    idleEmoji() {
      return this.data.emoji(this.emoji)
    },
  },

  watch: {
    searchText (data) {
      this.data.search(data, this.maxSearchResults)
    },
  },

  methods: {
    getItemprops (index) {
      const data = this.items[index]
      if (!data) {
        return
      }

      return {
        props: {
          value: data,
          native: this.native,
          skin: this.activeSkin,
          set: this.set,
          emojiTooltip: this.emojiTooltip,
          emojiSize: this.emojiSize,
        },
        on: {
          enter: this.onEmojiEnter,
          leave: this.onEmojiLeave,
          click: this.onEmojiClick,
        },
      }
    },

    onScrollUpdate (event, { start, end }) {
      if (this.skipScrollUpdate) {
        this.skipScrollUpdate = false
      } else {
        const data = this.items[end]

        if (data && data.category) {
          this.activeCategory = find(this.categories, item => item.id === data.category.id)
        } else if (Array.isArray(data) && data[0]) {
          this.activeCategory = findLast(this.categories, item => item.emojis.includes(data[0]))
        }
      }
    },

    onAnchorClick (category) {
      const idx = this.items.findIndex(item => (
        item.category &&
        item.category.id === category.id
      ))

      if (idx !== -1) {
        this.skipScrollUpdate = true
        this.startFromIdx = idx
        this.activeCategory = find(this.categories, item => item.id === category.id)
      }
    },

    onEmojiEnter (emoji) {
      this.previewEmoji = emoji
    },

    onEmojiLeave (emoji) {
      this.previewEmoji = null
    },

    onEmojiClick (emoji) {
      this.$emit('select', emoji)
      frequently.add(emoji)
    },

    onSkinChange (skin) {
      this.activeSkin = skin
      store.update({ skin })
      this.$emit('skin-change', skin)
    },
  },
}
</script>
