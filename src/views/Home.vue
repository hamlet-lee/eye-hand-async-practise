<template>
  <div class="home">
    <button v-if="state === BEFORE_START" @click="onStart">开始!</button>
    <div v-if="isRunning">
      <span v-for="(sec,secIdx) in sections">
        <span v-if="playingSecNo != secIdx">
          <span v-for="finger in sec">
            {{finger}}
          </span>
        </span>
        <span v-else>
          <span v-for="finger in sec">
            .
          </span>
        </span>
        |
      </span>
    </div>
    <div v-if="state === ENDING">
      <h2>游戏结束！</h2>
      <button @click="onConfirmEnd">OK!</button>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import st from '@/state.js'
export default {
  name: 'home',
  components: {
    // HelloWorld
  },
  timers: {
    running: {
      time: 1000,
      autostart: false,
      repeat: true
    }
  },
  data: function() {
    return {
      sections: [],
      playingSecNo: -1,
      state: st.BEFORE_START
    }
  },
  mounted () {

  },
  computed: {
    isRunning () {
      return this.state === st.RUNNING
    },
    BEFORE_START () {
      return st.BEFORE_START
    },
    ENDING () {
      return st.ENDING
    }
  },
  methods: {
    onStart() {
      this.state = st.RUNNING
      let sectionNum = 10
      let notesInSection = 4
      let fingerCount = 4
      this.createSections({sectionNum, notesInSection, fingerCount})
      this.playingSecNo = -1
      // https://www.npmjs.com/package/vue-timers
      this.$timer.start('running')
    },
    running () {
      console.log('Hello world' + new Date())
      if(this.playingSecNo >= this.sections.length) {
        this.$timer.stop('running')
        this.state = st.ENDING
      } else {
        this.playingSecNo++
      }
    },
    createSections({sectionNum, notesInSection, fingerCount}) {
      this.sections = []
      for(let i=0; i<sectionNum; i++) {
        let section = []
        for(let j=0; j<notesInSection; j++) {
          let note = Math.floor(Math.random() * fingerCount) + 1
          section.push(note)
        }
        this.sections.push(section)
      }
    },
    onConfirmEnd() {
      this.state = st.BEFORE_START
    }
  }
}
</script>
