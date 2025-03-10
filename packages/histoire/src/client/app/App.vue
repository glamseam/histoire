<script lang="ts">
export default {
  name: 'HistoireApp',
}
</script>

<script lang="ts" setup>
// @ts-expect-error virtual module
import { files as rawFiles, tree as rawTree, onUpdate } from '$histoire-stories'
import StoryList from './components/tree/StoryList.vue'
import BaseSplitPane from './components/base/BaseSplitPane.vue'
import { computed, onMounted, ref, watch } from 'vue'
import AppHeader from './components/app/AppHeader.vue'
import type { StoryFile, Tree } from './types'
import { useStoryStore } from './stores/story'
import { mapFile } from './util/mapping'
import { useTitle } from '@vueuse/core'
import { histoireConfig } from './util/config'
import { onKeyboardShortcut } from './util/keyboard'
import { isMobile } from './util/responsive'
import Breadcrumb from './components/app/Breadcrumb.vue'
import SearchModal from './components/search/SearchModal.vue'
import InitialLoading from './components/app/InitialLoading.vue'
import { MountStoryVue3 } from '@histoire/plugin-vue/client'

const files = ref<StoryFile[]>(rawFiles.map(file => mapFile(file)))
const tree = ref<Tree>(rawTree)

onUpdate((newFiles: StoryFile[], newTree: Tree) => {
  loading.value = false
  files.value = newFiles.map(file => {
    const existingFile = files.value.find(f => f.id === file.id)
    return mapFile(file, existingFile)
  })
  tree.value = newTree
})

const stories = computed(() => files.value.reduce((acc, file) => {
  acc.push(file.story)
  return acc
}, []))

// Store

const storyStore = useStoryStore()
watch(stories, value => {
  storyStore.setStories(value)
}, {
  immediate: true,
})

useTitle(computed(() => {
  if (storyStore.currentStory) {
    let title = storyStore.currentStory.title
    if (storyStore.currentVariant) {
      title += ` › ${storyStore.currentVariant.title}`
    }
    return `${title} | ${histoireConfig.theme.title}`
  }
  return histoireConfig.theme.title
}))

// Search

const loadSearch = ref(false)
const isSearchOpen = ref(false)

watch(isSearchOpen, value => {
  if (value) {
    loadSearch.value = true
  }
})

onKeyboardShortcut(['ctrl+k', 'meta+k'], (event) => {
  isSearchOpen.value = true
  event.preventDefault()
})

const loading = ref(false)

if (import.meta.hot && !rawFiles.length) {
  loading.value = true
  import.meta.hot.on('histoire:all-stories-loaded', () => {
    loading.value = false
  })
}

const mounted = ref(false)
onMounted(() => {
  mounted.value = true
})
</script>

<template>
  <div
    v-if="storyStore.currentStory"
    class="htw-hidden"
  >
    <MountStoryVue3
      :story="storyStore.currentStory"
    />
  </div>

  <div
    class="htw-h-screen htw-bg-white dark:htw-bg-gray-700 dark:htw-text-gray-100"
    :style="{
      // Prevent flash of content
      opacity: mounted && !loading ? 1 : 0,
    }"
  >
    <div
      v-if="isMobile"
      class="htw-h-full htw-flex htw-flex-col htw-divide-y htw-divide-gray-100 dark:htw-divide-gray-800"
    >
      <AppHeader @search="isSearchOpen = true" />
      <Breadcrumb
        :tree="tree"
        :stories="stories"
      />
      <RouterView class="htw-grow" />
    </div>

    <BaseSplitPane
      v-else
      save-id="main-horiz"
      :min="5"
      :max="50"
      :default-split="15"
      class="htw-h-full"
    >
      <template #first>
        <div class="htw-flex htw-flex-col htw-h-full">
          <AppHeader
            class="htw-flex-none"
            @search="isSearchOpen = true"
          />
          <StoryList
            :tree="tree"
            :stories="stories"
            class="htw-flex-1"
          />
        </div>
      </template>

      <template #last>
        <RouterView />
      </template>
    </BaseSplitPane>

    <SearchModal
      v-if="loadSearch"
      :shown="isSearchOpen"
      @close="isSearchOpen = false"
    />
  </div>

  <transition name="__histoire-fade">
    <InitialLoading
      v-if="loading"
    />
  </transition>
</template>
