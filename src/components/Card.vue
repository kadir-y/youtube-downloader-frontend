<template>
  <v-card class="mx-auto" :title="`#${props.index + 1} ${props.item.title}`"
    :color="props.item.selected ? 'green-darken-4' : 'red-darken-4'" :subtitle="props.item.uploader" width="%100"
    rounded="lg" link>
    <template v-slot:prepend>
      <v-avatar size="64" color="blue-darken-2" rounded="lg">
        <v-img :src="props.item.thumbnail.url"></v-img>
      </v-avatar>
    </template>
    <template v-slot:append>
      <v-checkbox v-model="props.item.selected"></v-checkbox>
    </template>
  </v-card>
</template>
<script setup>
const props = defineProps({
  item: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: false
  }
})
</script>