
<script setup>
import { onBeforeMount } from 'vue';
import { useSongStore } from './stores/song'
import { storeToRefs } from 'pinia';
const useSong = useSongStore()
const { isPlaying, currentTrack,  trackTime } = storeToRefs(useSong)

onBeforeMount(() => {
    isPlaying.value = false
    trackTime.value = '0:00'
})
</script>

<template>
    <TrackList />
    <Player v-if="currentTrack" />
</template>
