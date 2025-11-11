<template>
  <v-card class="mx-auto" :title="props.item.title"
    :subtitle="props.item.uploader" width="%100" link>
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
  }
})
</script>