<script setup>
import PlayerVolume from './PlayerVolume.vue'

import Play from 'vue-material-design-icons/Play.vue';
import Pause from 'vue-material-design-icons/Pause.vue';
import SkipBackward from 'vue-material-design-icons/SkipBackward.vue';
import SkipForward from 'vue-material-design-icons/SkipForward.vue';
import VolumeHigh from 'vue-material-design-icons/VolumeHigh.vue';
import VolumeMute from 'vue-material-design-icons/VolumeMute.vue';

import uniqolor from 'uniqolor';

import { useSongStore } from '~/stores/song'
import { storeToRefs } from 'pinia';
const useSong = useSongStore()
const { isPlaying, audio, currentTrack, currentArtist, trackTime, currentVolume } = storeToRefs(useSong)

let randColor = ref('')
randColor.value = uniqolor.random()
let isHover = ref(false)
let isVolumeHover = ref(false)
let isTrackTimeCurrent = ref('0:00')
let isTrackTimeTotal = ref('0:00')
let seeker = ref(null)
let seekerContainer = ref(null)
let range = ref(0)
let mouseOverTimeout = ref(null)

onMounted(() => {
    if (audio.value) {
        setTimeout(() => {
            timeupdate()
            loadmetadata()
        }, 300)
    }
    if (currentTrack.value) {
        seeker.value.addEventListener("change", function () {
            const time = audio.value.duration * (seeker.value.value / 100);
            audio.value.currentTime = time;
        });
        seeker.value.addEventListener("mousedown", function () {
            audio.value.pause();
            isPlaying.value = false
        });
        seeker.value.addEventListener("mouseup", function () {
            audio.value.play();
            isPlaying.value = true
        });
        seekerContainer.value.addEventListener("click", function (e) {
            const clickPosition = (e.pageX - seekerContainer.value.offsetLeft) / seekerContainer.value.offsetWidth;
            const time = audio.value.duration * clickPosition;
            audio.value.currentTime = time;
            seeker.value.value = (100 / audio.value.duration) * audio.value.currentTime;
        });
    }
})

const timeupdate = () => {
    audio.value.addEventListener("timeupdate", function () {
        var minutes = Math.floor(audio.value.currentTime / 60);
        var seconds = Math.floor(audio.value.currentTime - minutes * 60);
        isTrackTimeCurrent.value = minutes + ':' + seconds.toString().padStart(2, '0')
        trackTime.value = isTrackTimeCurrent.value
        const value = (100 / audio.value.duration) * audio.value.currentTime;
        range.value = value
        seeker.value.value = value;
    });
}
const loadmetadata = () => {
    audio.value.addEventListener('loadedmetadata', function () {
        const duration = audio.value.duration;
        const minutes = Math.floor(duration / 60);
        const seconds = Math.floor(duration % 60);
        isTrackTimeTotal.value = minutes + ':' + seconds.toString().padStart(2, '0')
    });
}
const mouseEnter = () => {
    if (mouseOverTimeout.value) {
        clearTimeout(mouseOverTimeout.value)
        mouseOverTimeout.value = null
    }
    isVolumeHover.value = true
}
const mouseLeave = () => {
    mouseOverTimeout.value = setTimeout(() => isVolumeHover.value = false, 100)
}
watch(() => audio.value, () => {
    timeupdate()
    loadmetadata()
})
watch(() => isTrackTimeCurrent.value, (time) => {
    if (time && time == isTrackTimeTotal.value) {
        useSong.nextSong(currentTrack.value)
    }
})
watch(() => currentTrack.value.id, (val) => {
    randColor.value = uniqolor.random()
})
</script>

<template>
    <div id="MusicPlayer" v-if="audio" class="
            fixed
            flex
            items-center
            justify-between
            bottom-0
            w-full
            z-50
            h-[80px]
            bg-[#23232D]
            border-t
            border-t-[#383838]
        ">
        <div class="flex items-center w-2/12">
            <div class="flex items-center justify-center h-[30px] pl-4">
                <button type="button" :disabled="currentTrack.id === 1"
                    :class="{ 'rounded-full hover:bg-[#363636]': currentTrack.id !== 1 }" class="mx-2 p-2"
                    @click="useSong.prevSong(currentTrack)">
                    <SkipBackward :fillColor="currentTrack.id === 1 ? '#747474' : '#FFFFFF'" :size="25" />
                </button>
                <button type="button" class="p-2 rounded-full hover:bg-[#363636]"
                    @click="useSong.playOrPauseThisSong(currentArtist, currentTrack)">
                    <Play v-if="!isPlaying" fillColor="#FFFFFF" :size="45" />
                    <Pause v-else fillColor="#FFFFFF" :size="45" />
                </button>
                <button type="button" class="mx-2 p-2 rounded-full hover:bg-[#363636]"
                    @click="useSong.nextSong(currentTrack)">
                    <SkipForward fillColor="#FFFFFF" :size="25" />
                </button>
            </div>
        </div>

        <div class=" mb-2.5 w-full max-w-[50%] mx-10">
            <div class="flex items-center justify-between pl-1 relative top-1 mx-7">
                <div class="flex items-center">
                    <div class="bg-[#2E2E39] py-0.5 px-1 text-[10px] text-[#72727D]">ALBUM</div>
                    <div class="text-white text-[14px] font-[300] ml-3">{{ currentTrack.name }}</div>
                    <div class="text-white relative -top-1 left-[6px]">.</div>
                    <div class="text-white text-[14px] font-[300] ml-3">{{ currentArtist.name }}</div>
                </div>

            </div>

            <div class="flex items-center">
                <div v-if="isTrackTimeCurrent" class="text-[#8a8a8a] text-[10px] pr-2 relative -bottom-[5px]">
                    {{ isTrackTimeCurrent }}
                </div>
                <div ref="seekerContainer" class="w-full relative mt-2 mb-3" @mouseenter="isHover = true"
                    @mouseleave="isHover = false">
                    <input v-model="range" ref="seeker" type="range" class="
                            absolute
                            rounded-full
                            my-[7px]
                            w-full
                            h-0
                            z-40
                            appearance-none
                            bg-opacity-100
                            focus:outline-none
                            cursor-pointer
                        " :class="{ 'rangeDotHidden': !isHover },
                            { 'rangeDot': isHover }
                            ">
                    <div class="pointer-events-none rounded-full absolute z-10 inset-y-0 left-0 w-0"
                        :style="`width: ${range}%; background-color: ${randColor.color}`"
                        :class="isHover ? 'h-[4px] mt-[5px]' : 'h-[2px] mt-[6px]'" />
                    <div :class="isHover ? 'h-[4px] mt-[5px]' : 'h-[2px] mt-[6px]'"
                        class="absolute z-[-0] inset-y-0 left-0 w-full bg-[#c4c4c4] rounded-full" />

                </div>
                <div v-if="isTrackTimeTotal" class="text-[#8a8a8a] text-[10px] pl-2 relative -bottom-[5px]">
                    {{ isTrackTimeTotal }}
                </div>
            </div>
        </div>

        <div class="flex justify-between items-center w-1/4 justify-end pr-6 ">
            <div class="flex items-center">
                <div @mouseenter="mouseEnter" @mouseleave="mouseLeave" class="relative">
                    <div class="p-2 hover:bg-[#5a5a5a] hover:bg-opacity-50 rounded-full cursor-pointer">
                        <VolumeHigh v-if="currentVolume > 0" class="block" fillColor="#FFFFFF" :size="17" />
                        <VolumeMute v-else class="block" fillColor="#FFFFFF" :size="17" />
                    </div>
                    <div v-show="isVolumeHover"
                        class="absolute -top-12 -left-20 p-2 px-4 bg-[#2a2a37] rounded-xl shadow-xl">
                        <PlayerVolume />
                    </div>
                </div>
            </div>
            <div class="flex items-center ml-1 ">
                <img class="rounded-sm" width="28" :src="currentArtist.albumCover">
                <div class="text-xs ml-1.5 text-white font-light">Queue</div>
            </div>
        </div>
    </div>
</template>

<style>
.rangeDotHidden[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 0;
    height: 0;
}

.rangeDot[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    background-color: white;
    border-radius: 100%;
    width: 12px;
    height: 12px;
}
</style>