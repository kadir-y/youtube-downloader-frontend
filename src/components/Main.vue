<template>
  <v-sheet class="mx-auto pa-5 mb-10" max-width="500" elevation="12" rounded="lg">
    <v-form validate-on="submit lazy" @submit.prevent="submit">
      <v-text-field v-model="link" :rules="rules" label="Paste youtube playlist or video link" density="compact"
        variant="outlined" prepend-inner-icon="mdi-magnify" single-line clearable>
      </v-text-field>
      <v-btn :loading="loading" class="mt-2" text="Search it" type="submit" variant="outlined" color="info"
        append-icon="mdi-arrow-right-thin" block></v-btn>
    </v-form>
  </v-sheet>

  <v-sheet v-if="items.type === 'video' || items.type === 'playlist'" class="mx-auto pa-5" max-width="700"
    elevation="12" rounded="lg">

    <v-sheet>
      <v-alert type="info" variant="outlined" class="mb-4" v-if="!isDownloading">
        <div>
          <strong>Info:</strong> Select videos and download them with your preferred settings.
        </div>
      </v-alert>
      <v-alert type="warning" variant="outlined" class="mb-4" v-if="isDownloading">
        <div>
          <strong>Downloading:</strong> {{ downloadStatus }}
        </div>
      </v-alert>
    </v-sheet>

    <v-row class="mb-2">
      <v-col>
        <span class="text-h6">
          {{ items.title }}
        </span>
      </v-col>
      <v-col class="d-flex justify-end">
        <v-btn class="mr-2" variant="outlined" color="info" size="large" @click="selectAll">
          {{ allSelected ? 'Deselect All' : 'Select All' }}
        </v-btn>
        <v-btn variant="outlined" color="success" size="large" @click="downloadSelected" 
          :disabled="!hasSelected || isDownloading" :loading="isDownloading">
          Download
        </v-btn>
      </v-col>
    </v-row>

    <v-sheet class="mb-6">
      <p class="mb-2">Quick Settings</p>
      <v-row>
        <v-col cols="auto">
          <v-select v-model="selectedQuality" label="Quality" :items="['best', 'good', 'worst']" hide-details density="compact"></v-select>
        </v-col>
        <v-col cols="auto">
          <v-select v-model="selectedType" label="Type" :items="['video', 'audioonly']" hide-details density="compact"></v-select>
        </v-col>
        <v-col class="d-flex align-center" cols="auto">
          <v-progress-circular v-if="isDownloading" :value="downloadProgress" size="40" width="3" color="success">
            {{ downloadProgress }}%
          </v-progress-circular>
          <span v-else-if="downloadProgress > 0" class="ml-2">{{ downloadProgress }}%</span>
        </v-col>
      </v-row>
    </v-sheet>

    <template v-for="(item, index) in items.videos" :key="item.id">
      <Card class="mb-3" :item="item" :index="index"></Card>
    </template>

  </v-sheet>
</template>

<script setup>
import { ref, watch, computed } from 'vue'
import axios from 'axios'
import Card from './Card.vue'

const rules = [value => checkApi(value)]

const loading = ref(false)
const link = ref('')
const items = ref({})
const selectedQuality = ref('best')
const selectedType = ref('video')
const isDownloading = ref(false)
const downloadProgress = ref(0)
const downloadStatus = ref('')

const hasSelected = computed(() => {
  return items.value.videos && items.value.videos.some(v => v.selected)
})

const allSelected = computed(() => {
  return items.value.videos && items.value.videos.length > 0 && items.value.videos.every(v => v.selected)
})

async function submit(event) {
  loading.value = true
  const results = await event
  loading.value = false
}

async function checkApi(link) {
  if (!link) {
    return "Link is required."
  }
  try {
    const response = await axios.get(`http://localhost:3000/api/get-metadata?link=${link}`)
    if (response.data.error) {
      return "An error occurred while fetching metadata."
    }
    items.value = response.data.list
    // Initialize selected property
    if (items.value.videos) {
      items.value.videos.forEach(video => {
        video.selected = false
      })
    }
    return true
  } catch (error) {
    console.error(error)
    return "An error occurred while fetching metadata."
  }
}

const selectAll = () => {
  if (items.value.videos) {
    const shouldSelect = !allSelected.value
    items.value.videos.forEach(video => {
      video.selected = shouldSelect
    })
  }
}

const downloadSelected = async () => {
  const selected = items.value.videos.filter(v => v.selected)
  if (selected.length === 0) return

  isDownloading.value = true

  for (let i = 0; i < selected.length; i++) {
    const video = selected[i]
    await downloadVideo(video, i + 1, selected.length)
  }

  isDownloading.value = false
  downloadProgress.value = 0
  downloadStatus.value = ''
}

const downloadVideo = (video, current, total) => {
  return new Promise((resolve, reject) => {
    downloadStatus.value = `Downloading ${current}/${total}: ${video.title}`
    downloadProgress.value = 0

    const eventSource = new EventSource(
      `http://localhost:3000/api/download?link=${encodeURIComponent(video.url)}&quality=${selectedQuality.value}&type=${selectedType.value}`
    )

    eventSource.onmessage = (event) => {
      try {
        const progress = JSON.parse(event.data)
        console.log('Progress data:', progress)
        
        // yt-dlp'den gelen progress verisi
        if (progress.percentage !== undefined) {
          downloadProgress.value = Math.round(progress.percentage)
          downloadStatus.value = `Downloading ${current}/${total}: ${video.title} - ${downloadProgress.value}% (${progress.speed_str})`
          console.log('Updated downloadProgress:', downloadProgress.value)
        }
      } catch (e) {
        console.error('Error parsing progress:', e)
      }
    }

    eventSource.addEventListener('done', (event) => {
      try {
        const data = JSON.parse(event.data)
        downloadProgress.value = 100
        downloadStatus.value = `Completed: ${video.title}`
        console.log('Download complete')

        // Dosyayı indir
        setTimeout(() => {
          window.location.href = `http://localhost:3000/api/download-file/${data.file}`
        }, 500)

        eventSource.close()
        setTimeout(resolve, 1500)
      } catch (e) {
        console.error('Error handling done event:', e)
        eventSource.close()
        resolve()
      }
    })

    eventSource.onerror = (error) => {
      console.error('Download error:', error)
      downloadStatus.value = `Error: Download failed`
      eventSource.close()
      reject(error)
    }
  })
}

watch(items, (newItems) => {
  console.log("Items updated:", newItems)
})
</script>