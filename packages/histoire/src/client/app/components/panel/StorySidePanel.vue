<script lang="ts" setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useStoryStore } from '../../stores/story'

import BaseSplitPane from '../base/BaseSplitPane.vue'
import BaseEmpty from '../base/BaseEmpty.vue'
import StoryControls from './StoryControls.vue'
import StoryDocs from './StoryDocs.vue'
import StoryEvents from './StoryEvents.vue'
import StorySourceCode from './StorySourceCode.vue'
import PaneTabs from './PaneTabs.vue'

const storyStore = useStoryStore()

const route = useRoute()

const panelContentComponent = computed(() => {
  switch (route.query.tab) {
    case 'docs':
      return StoryDocs
    case 'events':
      return StoryEvents
    default:
      return StoryControls
  }
})
</script>

<template>
  <BaseEmpty v-if="!storyStore.currentVariant">
    <span>Select a variant</span>
  </BaseEmpty>

  <BaseEmpty v-else-if="!storyStore.currentVariant.configReady || !storyStore.currentVariant.previewReady">
    <span>Loading...</span>
  </BaseEmpty>

  <BaseSplitPane
    v-else
    save-id="story-sidepane"
    orientation="portrait"
    class="htw-h-full"
    data-test-id="story-side-panel"
  >
    <template #first>
      <div class="htw-flex htw-flex-col htw-h-full">
        <PaneTabs
          :story="storyStore.currentStory"
          :variant="storyStore.currentVariant"
        />

        <component
          :is="panelContentComponent"
          :story="storyStore.currentStory"
          :variant="storyStore.currentVariant"
          class="htw-h-full htw-overflow-auto"
        />
      </div>
    </template>

    <template #last>
      <StorySourceCode
        :variant="storyStore.currentVariant"
        class="htw-h-full"
      />
    </template>
  </BaseSplitPane>
</template>
