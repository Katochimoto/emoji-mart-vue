<template>

<div :class="{ 'emoji-mart-category': true, 'emoji-mart-no-results': !hasResults }" v-if="isVisible && (isSearch || hasResults)">
  <div class="emoji-mart-category-label">
    <span>{{ i18n.categories[id] }}</span>
  </div>

  <template v-for="{ emojiObject, emojiView} in emojiObjects">
    <span
      v-if="emojiView.canRender"
      :data-title="emojiObject.short_name"
      :title="emojiView.title"
      class="emoji-mart-emoji"
      @mouseenter="emojiProps.onEnter(emojiView.getEmoji())"
      @mouseleave="emojiProps.onLeave(emojiView.getEmoji())"
      @click="emojiProps.onClick(emojiView.getEmoji())">
      <span  :class="emojiView.cssClass" :style="emojiView.cssStyle">{{emojiView.content}}</span>
    </span>
  </template>

  <div v-if="!hasResults">
    <nimble-emoji
      :data="data"
      emoji="sleuth_or_spy"
      :native="emojiProps.native"
      :skin="emojiProps.skin"
      :set="emojiProps.set"
    />
    <div class="emoji-mart-no-results-label">{{ i18n.notfound }}</div>
  </div>
</div>

</template>

<script>

import { EmojiView } from '../utils/emoji-data'
import NimbleEmoji from './emoji/nimbleEmoji'


export default {
  props: {
    data: {
      type: Object,
      required: true
    },
    i18n: {
      type: Object,
      required: true
    },
    id: {
      type: String,
      required: true
    },
    name: {
      type: String,
      required: true
    },
    emojis: {
      type: Array
    },
    emojiProps: {
      type: Object,
      required: true
    }
  },
  computed: {
    isVisible() {
      return !!this.emojis
    },
    isSearch() {
      return this.name == 'Search'
    },
    hasResults() {
      return this.emojis.length > 0
    },
    emojiObjects() {
      return this.emojis.map((emoji) => {
          let emojiObject = emoji
          let emojiView = new EmojiView(
            emoji,
            this.emojiProps.skin,
            this.emojiProps.set,
            this.emojiProps.native,
            this.emojiProps.fallback,
            this.emojiProps.emojiTooltip,
            this.emojiProps.emojiSize,
          )
          return { emojiObject, emojiView }
      })
    }
  },
  components: {
    NimbleEmoji
  }
}

</script>
