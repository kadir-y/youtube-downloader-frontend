<template>
  <v-sheet class="mx-auto pa-5 mb-10" max-width="500" elevation="12" rounded="lg">
    <v-form validate-on="submit lazy" @submit.prevent="submit">
      <v-text-field v-model="link" :rules="rules" label="Paste youtube playlist or video link" density="compact"
        variant="outlined" prepend-inner-icon="mdi-magnify" single-line clearable>
      </v-text-field>
      <v-btn :loading="loading" class="mt-2" text="Search it" type="submit" variant="outlined" color="info" append-icon="mdi-arrow-right-thin" block></v-btn>
    </v-form>
  </v-sheet>

  <v-sheet v-if="items.type === 'video' || items.type === 'playlist'" class="mx-auto pa-5" max-width="700" elevation="12" rounded="lg">

    <template v-if="items.type === 'video'">
      <Card :item="items"></Card>
    </template>
  
    <template v-if="items.type === 'playlist'">
      <v-row class="mb-2">
        <v-col>
          <span class="text-h6">
            {{ items.title }}
          </span>
        </v-col>
        <v-col class="d-flex justify-end">
          <v-btn variant="outlined" color="success" disabled>Download</v-btn>
        </v-col>
      </v-row>
      <Card v-for="item in items.videos" :key="item.id" :item="item" class="mb-4"></Card>
    </template>

  </v-sheet>
</template>

<script setup>
import { ref, watch } from 'vue'
import axios from 'axios'

const rules = [value => checkApi(value)]

const loading = ref(false)
const link = ref('')
const items = ref({})

function handleClick () {
  alert("foo")
}

async function submit(event) {
  loading.value = true
  const results = await event
  loading.value = false
  alert(JSON.stringify(results, null, 2))
}

async function checkApi(link) {
  if (!link) {
    return "Link is required."
  }
  try {
    const response = await axios.get(`http://localhost:3000/api/get-metadata?link=${link}`);
    if (response.data.error) {
      return "An error occurred while fetching metadata."
    }
    items.value = response.data.list
    return true
  } catch (error) {
    console.error(error)
    return "An error occurred while fetching metadata."
  }

}

  watch(items, (newitems) => {
    console.log("Items updated:", newitems)
  })
</script>