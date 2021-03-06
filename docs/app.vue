<template>
  <div>
    <div class="row">
      <h1>Emoji Mart Vue 🏬</h1>
    </div>
    <div class="row">
      <emoji emoji=":santa::skin-tone-3:" :size="32" />
      <emoji emoji="santa" set="emojione" :size="32" />
      <emoji :emoji="santaEmojiObject" :size="32" />
    </div>

    <div class="row">
      <button @click="toggleVisible">Show / hide the picker</button>
      <button @click="toggleVisible" v-html="smile"></button>
    </div>
    <div class="row">
      <template
        v-for="set in [
          'native',
          'apple',
          'google',
          'twitter',
          'emojione',
          'messenger',
          'facebook',
        ]"
      >
        <button
          :key="set"
          @click="activeSet = set"
          :disabled="activeSet == set"
        >
          {{ set }}
        </button>
      </template>
    </div>

    <div v-if="isVisible" class="row">
      <nimble-picker
        :data="index"
        :set="activeSet"
        :native="native"
        :custom="custom"
        :emoji="emoji"
        :emojiTooltip="true"
        :title="title"
      />
    </div>

    <div class="row-small">
      <iframe
        src="https://ghbtns.com/github-btn.html?user=jm-david&repo=emoji-mart-vue&type=star&count=true"
        frameBorder="0"
        scrolling="0"
        width="90px"
        height="20px"
      ></iframe>
    </div>

    <div class="row"></div>
    <div class="row">
      <div>Custom Search And Preview Templates Example</div>
      <Picker :native="true">
        <template slot="searchTemplate" slot-scope="slotProps">
          <input
            type="text"
            :placeholder="slotProps.i18n.search"
            v-bind="slotProps"
          />
        </template>
        <template slot="previewTemplate" slot-scope="slotProps">
          <div class="emoji-mart-preview">
            <div class="emoji-mart-preview-emoji">
              <NimbleEmoji
                :data="slotProps.data"
                :emoji="slotProps.emoji ? slotProps.emoji : 'point_up'"
                :native="slotProps.emojiProps.native"
                :skin="slotProps.emojiProps.skin"
                :set="slotProps.emojiProps.set"
              />
            </div>
            <div class="emoji-mart-preview-data">
              <div class="emoji-mart-preview-name">
                {{
                  slotProps.emoji
                    ? '~' + slotProps.emoji.name + '~'
                    : 'Choose...'
                }}
              </div>
              <div class="emoji-mart-preview-shortnames">
                {{ slotProps.emoji ? slotProps.emoji.colons : '' }}
              </div>
            </div>
          </div>
        </template>
      </Picker>
    </div>

    <div class="row">
      <div>Filtered picker example</div>
      <NimblePicker
        :native="true"
        emoji="flag-tf"
        :emojiSize="18"
        :data="indexFiltered"
      />
    </div>
  </div>
</template>

<script>
import data from '../data/all.json'
import { Picker, NimblePicker, NimbleEmoji, Emoji, EmojiIndex } from '../src'
import '../css/emoji-mart.css'

const CUSTOM_EMOJIS = [
  {
    name: 'Party Parrot',
    short_names: ['parrot'],
    keywords: ['party'],
    imageUrl: 'images/parrot.gif',
  },
  {
    name: 'Octocat',
    short_names: ['octocat'],
    keywords: ['github'],
    imageUrl: 'https://github.githubassets.com/images/icons/emoji/octocat.png',
  },
  {
    name: 'Squirrel',
    short_names: ['shipit', 'squirrel'],
    keywords: ['github'],
    imageUrl: 'https://github.githubassets.com/images/icons/emoji/shipit.png',
  },
]

let index = new EmojiIndex(data, {
  custom: CUSTOM_EMOJIS,
})

let indexFiltered = new EmojiIndex(data, {
  custom: CUSTOM_EMOJIS,
  emojisToShowFilter: (emoji) => {
    return emoji.short_names[0].match(/^flag.*/) !== null
  },
})

export default {
  data() {
    return {
      index: index,
      indexFiltered: indexFiltered,
      activeSet: 'native',
      emoji: 'point_up',
      title: 'Pick your emoji…',
      custom: CUSTOM_EMOJIS,
      isVisible: true,
    }
  },
  computed: {
    native() {
      return this.activeSet == 'native'
    },
    smile() {
      // Static emoji example
      let emoji = index.findEmoji(':smile:')
      // Note, that position in the emoji sheet is calculated by
      // `emoji` object
      let style = `background-position: ${emoji.getPosition()}; background-image: url(https://unpkg.com/emoji-datasource-emojione@4.0.4/img/emojione/sheets-256/64.png); width: 24px; height: 24px; display: inline-block; background-size: 5200%`
      return `<div class='emoji' style="${style}"></div>`
    },
    santaEmojiObject() {
      return index.findEmoji(':santa:')
    },
  },
  methods: {
    toggleVisible() {
      this.isVisible = !this.isVisible
    },
  },
  components: {
    Picker,
    NimblePicker,
    Emoji,
    NimbleEmoji,
  },
}
</script>

<style scoped>
button + button {
  margin-left: 0.5em;
}
button {
  padding: 0.4em 0.6em;
  border-radius: 5px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  background: #fff;
  outline: 0;
  cursor: pointer;
}

button[disabled] {
  border-color: #ae65c5;
  cursor: default;
}

h1 {
  font-family: Courier;
}

.row + .row {
  margin-top: 2em;
}

.row-small {
  margin-top: 1em;
}

.emoji-mart {
  margin: 0 auto;
  text-align: left;
}

.emoji-mart-title-label {
  font-size: 21px;
}
</style>
