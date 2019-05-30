<template>
  <q-page class="bg-dark column window-height">

    <!-- Cabeçalho -->
    <div class="col-2 q-pa-md" style="text-align: center;">
      <div class="q-title text-weight-light text-blue-grey-2">
        Castelo Forte
      </div>
      <transition name="slide-fade" mode="out-in">
        <p class="q-subheading text-weight-light text-blue-grey-2" :key="currentSong">
          {{ musicPlaylist[currentSong] !== undefined ? musicPlaylist[currentSong].date : '' }}
        </p>
      </transition>
    </div>

    <!-- Imagem -->
    <div class="col-4" style="text-align: center; overflow: hidden;">
        <transition name="ballmove" enter-active-class="animated zoomIn" leave-active-class="animated fadeOutDown" mode="out-in">
          <img :src="musicPlaylist[currentSong] !== undefined ? musicPlaylist[currentSong].image : ''" :key="currentSong" style="width: 200px;" class="shadow-8">
        </transition>
    </div>

    <!-- Versículo -->
    <div class="col-3" style="text-align: center;">
      <transition name="slide-fade" mode="out-in">
          <p class="q-display-1 text-weight-thin text-blue-grey-2 q-mt-xl title" :key="currentSong">
            {{ musicPlaylist[currentSong] !== undefined ? musicPlaylist[currentSong].title : '' }}
          </p>
        </transition>
    </div>

    <!-- Footer -->
    <div class="col">
      <div class="row">
        <div class="col-12 q-px-lg">
          <q-slider
            :value="currentTime"
            :label-value="currentTime | fancyTimeFormat"
            :min="0"
            :max="trackDuration"
            color="light-blue-5"
            snap
            @change="val => { currentTime = val; audio.currentTime = val; }"
          />
        </div>
      </div>
      <div class="row">
        <div class="col-6 text-blue-grey-2 q-px-lg" style="text-align: left;">
          {{ currentTime | fancyTimeFormat }}
        </div>
        <div class="col-6 text-blue-grey-2 q-px-lg" style="text-align: right;">
          {{ trackDuration | fancyTimeFormat }}
        </div>
      </div>
      <div class="row">
        <div class="col-4" style="text-align: center;">
          <q-btn
            v-on:click="prevSong()"
            icon="skip_previous"
            color="light-blue-5"
            size="xl"
            round
            flat
            :disable="currentSong === 0"
          />
        </div>
        <div class="col-4" style="text-align: center;">
          <q-btn
            v-on:click="playAudio()"
            :icon="iconButtonPlay"
            color="light-blue-5"
            size="xl"
            round
            outline
          />
        </div>
        <div class="col-4" style="text-align: center;">
          <q-btn
            v-on:click="nextSong()"
            icon="skip_next"
            color="light-blue-5"
            size="xl"
            round
            flat
            :disable="currentSong === musicPlaylist.length-1"
          />
        </div>
      </div>
    </div>

  </q-page>
</template>

<style>
.animated {
  animation-duration: 0.5s;
}
/* data change transitions */
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.2s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter,
.slide-fade-leave-to {
  transform: translateY(10px);
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
.title {
  transition-delay: 100ms;
}
img {
  width: 100%;
  height: 100%;
  z-index: 10;
  object-fit: cover;
  object-position: 50% 50%;
}
</style>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      selectedValue: 0,
      audio: '',
      imgLoaded: false,
      currentlyPlaying: false,
      currentlyStopped: false,
      currentTime: 0,
      checkingCurrentPositionInTrack: '',
      trackDuration: 0,
      currentProgressBar: 0,
      isPlaylistActive: false,
      currentSong: 0,
      debug: false,
      musicPlaylist: [],
      audioFile: ''
    }
  },
  filters: {
    fancyTimeFormat: function (s) {
      if (s !== undefined) {
        let totalSeconds = s
        let hours = Math.floor(totalSeconds / 3600)
        totalSeconds %= 3600
        let minutes = Math.floor(totalSeconds / 60)
        let seconds = totalSeconds % 60

        minutes = String(minutes).padStart(2, '0')
        hours = String(hours).padStart(2, '0')
        seconds = String(Math.floor(seconds)).padStart(2, '0')
        return hours + ':' + minutes + ':' + seconds
      } else {
        return ''
      }
    }
  },
  computed: {
    iconButtonPlay: function () {
      return this.currentlyPlaying ? 'pause' : 'play_arrow'
    }
  },
  methods: {
    togglePlaylist: function () {
      this.isPlaylistActive = !this.isPlaylistActive
    },
    nextSong: function () {
      if (this.currentSong < this.musicPlaylist.length - 1) {
        this.changeSong(this.currentSong + 1)
      }
    },
    prevSong: function () {
      if (this.currentSong > 0) {
        this.changeSong(this.currentSong - 1)
      }
    },
    changeSong: function (index) {
      // if (this.musicPlaylist[this.currentSong] !== undefined) {
      var wasPlaying = this.currentlyPlaying
      this.imageLoaded = false
      if (index !== undefined) {
        this.stopAudio()
        this.currentSong = index
      }
      this.audioFile = this.musicPlaylist[this.currentSong].url
      this.audio = new Audio(this.audioFile)
      var localThis = this
      this.audio.addEventListener('loadedmetadata', function () {
        localThis.trackDuration = Math.round(this.duration)
      })
      this.audio.addEventListener('ended', this.handleEnded)
      if (wasPlaying) {
        this.playAudio()
      }
      // }
    },
    isCurrentSong: function (index) {
      if (this.currentSong === index) {
        return true
      }
      return false
    },
    getCurrentSong: function (currentSong) {
      return this.musicPlaylist[currentSong].url
    },
    playAudio: function () {
      if (
        this.currentlyStopped === true &&
        this.currentSong + 1 === this.musicPlaylist.length
      ) {
        this.currentSong = 0
        this.changeSong()
      }
      if (!this.currentlyPlaying) {
        this.getCurrentTimeEverySecond(true)
        this.currentlyPlaying = true
        this.audio.play()
      } else {
        this.stopAudio()
      }
      this.currentlyStopped = false
    },
    stopAudio: function () {
      this.audio.pause()
      this.currentlyPlaying = false
      this.pausedMusic()
    },
    handleEnded: function () {
      if (this.currentSong + 1 === this.musicPlaylist.length) {
        this.stopAudio()
        this.currentlyPlaying = false
        this.currentlyStopped = true
      } else {
        this.currentlyPlaying = false
        this.currentSong++
        this.changeSong()
        this.playAudio()
      }
    },
    onImageLoaded: function () {
      this.imgLoaded = true
    },
    getCurrentTimeEverySecond: function (startStop) {
      var localThis = this
      this.checkingCurrentPositionInTrack = setTimeout(
        function () {
          localThis.currentTime = localThis.audio.currentTime
          localThis.currentProgressBar = localThis.audio.currentTime / localThis.trackDuration * 100
          localThis.getCurrentTimeEverySecond(true)
        },
        1000
      )
    },
    pausedMusic: function () {
      clearTimeout(this.checkingCurrentPositionInTrack)
    }
  },
  created: function () {
    this.$axios.get('https://alertamundo.000webhostapp.com/')
      .then((response) => {
        this.musicPlaylist = response.data
        // this.$q.notify({
        //   color: 'positive',
        //   position: 'top',
        //   message: 'Sucesso ao carregar mensagens',
        //   icon: 'report_problem'
        // })
        this.changeSong()
        this.audio.loop = false
      })
      .catch((e) => {
        this.$q.notify({
          color: 'negative',
          position: 'top',
          message: 'Falha ao carregar mensagens. ' + e,
          icon: 'report_problem'
        })
      })
  }
}
</script>
