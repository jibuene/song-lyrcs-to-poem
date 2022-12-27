<route lang="json">
{
  "meta": {
    "title": "Home"
  }
}
</route>

<script setup lang="ts">
  import { ref, onMounted, watch, nextTick, reactive } from 'vue'

  import axios from 'axios'
  import Swal from 'sweetalert2'
  import { useHead } from '@vueuse/head';

  useHead({
    title: 'Home'
  })

  const textBox = ref(null);

  const state = reactive({
    title: '',
    lyric: '',
    artist: '',
    fontsList: [],
    customFont: '',
    lyricIdx: 0,
    loading: false,
    img: {
      width: 1200,
      height: 880
    }
  })

  const randomIntFromInterval = (min: number, max: number) => { 
    return Math.floor(Math.random() * (max - min + 1) + min)
  }

  const getLyrics = async (artist: String) => {
    try {
      const response = await axios.get('http://localhost:5000/api/v1/lyrics/search/' + artist)
      console.log(response.data)
      // Grab title
      state.title = response.data.split('Lyrics')[0]
      response.data = response.data.split('Lyrics')[1]

      // Filter out useless lyrics and filler
      response.data = response.data.replace(/ *\[[^\]]*]/g, 'splithere')
      response.data = response.data.replace(/ *\[(^\))*]/g, 'splithere')
      response.data = response.data.replaceAll('\x0a\x0a', 'splithere')
      response.data = response.data.replaceAll('You might also like', '')
      response.data = response.data.replaceAll('You might also like', '')
      response.data = response.data.replaceAll('Embed', '')
  
      const lyrics = response.data.split('splithere').filter((x: string) => x.length > 20)
      const randIndex = randomIntFromInterval(0, lyrics.length)
      state.lyricIdx = randIndex
      state.lyric = lyrics
    } catch (error:any) {
      Swal.fire({
        title: error.response.data,
        text: 'Try again or change the artist',
        icon: 'error'
      })

    }
      }


  // Simple script to random choose google font
  // Not my API key. Found on codepen lol
  const API_KEY = 'AIzaSyDDiO8nLVRMDaXwrJp61Cdcar5gFmmiR1Q';
  const loadFontsList = async () => {
    try {
      const result = await fetch('https://www.googleapis.com/webfonts/v1/webfonts?key=' + API_KEY)
      const data = await result.json()
      console.log('Loaded google fonts list: ', data.items.length)
      return data.items
    } catch (error) {
      console.log('loadFontsList', error)
    }
  }
  const loadRandomFont = async () => {
    const randomIndex = Math.floor(Math.random() * state.fontsList.length);
    const chosenFont = state.fontsList[randomIndex].family;
    WebFont.load({
      google: {
        families: [chosenFont]
      }
    })
    console.log('Chosen font: ', chosenFont);
    state.customFont = 'font-family: ' + chosenFont + ';'
  }

  watch(
    () => [state.lyricIdx, state.customFont],
    async () => {
      await nextTick()
      if (textBox.value !== null) {
        if (state.img.width !== textBox.value.clientWidth + 110 && state.img.height !== textBox.value.clientHeight + 70) {
          state.loading = true
        }
        console.log(`Changed idx. Width of div is: ${textBox.value.clientWidth} height: ${textBox.value.clientHeight}`)
        state.img.width = textBox.value.clientWidth + 110
        state.img.height = textBox.value.clientHeight + 70
      }

    }
  )

  onMounted(async () => {
    state.fontsList = await loadFontsList();
    loadRandomFont()
  })

</script>

<template>
  <div class="bg-base-100">
    <div class="form-control center">
      <h1 class="font-medium leading-tight text-4xl mt-0 mb-2">Search for artist</h1>
      <div class="input-group center">
        <input type="text" placeholder="Search…" class="input input-bordered w-3/6" v-model="state.artist" />
        <button class="btn btn-square" @click="getLyrics(state.artist)">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
        </button>
      </div>
    </div>
    <div v-if="state.lyric.length !== 0">
      <div class="form-control center mb-5">
        <h1 class="font-medium leading-tight text-4xl mt-0 mb-2">{{ state.title }}</h1>
        <label class="label">
          <span class="label-text">Chose verse</span>
        </label>
        <select class="select select-bordered w-full max-w-xs" v-model="state.lyricIdx">
          <option v-for="idx in state.lyric.length" :key="idx">{{ idx - 1 }}</option>
        </select>

      </div>

      <img id="image" :src="'https://picsum.photos/' + state.img.width+'/'+state.img.height+'/?blur'" @load="state.loading = false" :hidden="state.loading" />
      <p id="text" style="white-space: pre-line;" :style="state.customFont" ref="textBox" :hidden="state.loading">
        {{ state.lyric[state.lyricIdx] }}
      </p>
      <div class="form-control w-full max-w-xs">
        <button class="btn btn-primary" @click="loadRandomFont()">Change font!</button>
      </div>
    </div>

  </div>
</template>

<style scoped>
  #text {
    /* White on black text magic */
    position: absolute;
    color: white;
    text-shadow: 1px 1px 0 #000,
    -1px 1px 0 #000,
    1px -1px 0 #000,
    -1px -1px 0 #000,
    0px 1px 0 #000,
    0px -1px 0 #000,
    -1px 0px 0 #000,
    1px 0px 0 #000,
    2px 2px 0 #000,
    -2px 2px 0 #000,
    2px -2px 0 #000,
    -2px -2px 0 #000,
    0px 2px 0 #000,
    0px -2px 0 #000,
    -2px 0px 0 #000,
    2px 0px 0 #000,
    1px 2px 0 #000,
    -1px 2px 0 #000,
    1px -2px 0 #000,
    -1px -2px 0 #000,
    2px 1px 0 #000,
    -2px 1px 0 #000,
    2px -1px 0 #000,
    -2px -1px 0 #000;
    /* background: rgba(51, 170, 51,  0.4); */
    border-radius: 5px;
    font-size: 34px;
    font-weight: bold;
    left: 50px;
    top: 250px;
  }

  .center {
    display: flex;
    justify-content: center;
    align-items: center;
  }

</style>