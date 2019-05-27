<template>
  <div>
    <div
      v-if="value.category"
      class="emoji-mart-category-label"
    >
      {{ value.category.name }}
    </div>
    <div
      v-else
      class="emoji-mart-emoji-group"
    >
      <div
        v-for="item in emojis"
        :key="item.id"
        :title="item.title"
        class="emoji-mart-emoji"
        @mouseenter="$emit('enter', item.getEmoji())"
        @mouseleave="$emit('leave', item.getEmoji())"
        @click="$emit('click', item.getEmoji())"
      >
        <div
          :class="{
            'emoji-mart-emoji-content': true,
            [ item.cssClass ]: true
          }"
          :style="item.cssStyle"
        >
          {{ item.content }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { EmojiView } from '../utils/emoji-data'

export default {
  props: {
    value: {
      type: [ Object, Array ],
      required: true,
    },
    native: {},
    skin: {},
    set: {},
    fallback: {},
    emojiTooltip: {},
    emojiSize: {},
  },

  computed: {
    emojis () {
      if (!Array.isArray(this.value)) {
        return []
      }

      return this.value.map(emoji => new EmojiView(
        emoji,
        this.skin,
        this.set,
        this.native,
        this.fallback,
        this.emojiTooltip,
        this.emojiSize,
      ))
    },
  },
}
</script>
