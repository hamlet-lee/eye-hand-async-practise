<template>
  <div class="home">
    <div v-if="state === BEFORE_START">
      每分钟<input v-model="spm"></input>小节
      <button @click="onStart">开始</button>
    </div>
    <div v-if="isRunning">
      <span v-for="(sec,secIdx) in sections">
        <template v-if="playingSecNo > secIdx">
          <span class="note" v-for="note in sec">
            <span v-bind:class="{'succ': note.status === 'SUCCESS', 'fail': note.status === 'MISSED'}">
              {{note.finger}}
            </span>
          </span>
        </template>
        <template v-else-if="playingSecNo == secIdx">
          <span class="note" v-for="note in sec">
            <span class="succ" v-if="note.status === 'SUCCESS'">{{note.finger}}</span>
            <span v-if="note.status === 'NOT_HIT'">.</span>
          </span>
        </template>
        <template v-else-if="secIdx > playingSecNo && secIdx <= playingSecNo + coveredSecNum">
          <span class="note" v-for="note in sec">
            x
          </span>
        </template>
        <template v-else-if="playingSecNo + coveredSecNum + 1 == secIdx">
          <span class="note looking-ahead" v-for="note in sec">
            {{note.finger}}
          </span>
        </template>
        <template v-else>
          <span class="note" v-for="note in sec">
            {{note.finger}}
          </span>
        </template>
        |
      </span>
    </div>
    <div v-if="state === RUNNING || state === ENDING ">
      正确：{{correctNum}}
    </div>
    <div v-if="state === ENDING">
      <h2>游戏结束！得分{{score}}</h2>
      <button @click="onConfirmEnd">好的</button>
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
      time: 3000,
      autostart: false,
      repeat: true
    }
  },
  data: function() {
    return {
      spm: 30,           // 每分钟多少小节
      sections: [],      // “乐谱”
      playingSecNo: -1,  // 当前在演奏第几个小节
      state: st.BEFORE_START, // 当前状态
      coveredSecNum: 1,  // 要求提前看几个小节？
      posInPlayingSec: 0, // 当前在演奏第几个音符
      correctNum: 0       // 本局演奏正确的音符数
    }
  },
  created: function(){
      var _this = this;
      window.onkeydown = (e) => {
        this.onPress(e.key)
    }
  },
  mounted () {

  },
  computed: {
    score() {
      let s = 0
      for(let i=0; i<this.sections.length; i++) {
        s += this.sections[i].length
      }
      return Math.floor(this.correctNum / s * 100)
    },
    isRunning () {
      return this.state === st.RUNNING
    },
    RUNNING () {
      return st.RUNNING
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
      let sectionNum = 40
      let notesInSection = 2
      let fingerCount = 4
      this.createSections({sectionNum, notesInSection, fingerCount})
      this.playingSecNo = -1 - this.coveredSecNum
      this.correctNum = 0
      this.posInPlayingSec = 0
      this.state = st.RUNNING
      // https://www.npmjs.com/package/vue-timers
      this.timers.running.time = 1000 / (this.spm / 60)
      this.$timer.start('running')
    },
    onPress(key) {
      console.log(`key=${key}`)
      if( this.state == st.RUNNING) {
        if(this.playingSecNo >= 0) {
          let playingSec = this.sections[this.playingSecNo]
          if(this.posInPlayingSec < playingSec.length) {
            let expectNote = playingSec[this.posInPlayingSec]
            console.log(`expectNote=${expectNote}`)
            if(expectNote.finger == key ) {
              this.correctNum ++
              this.posInPlayingSec ++
              expectNote.status = 'SUCCESS'
            }
          }
        }
      }
    },
    running () {
      console.log('Hello world' + new Date())
      if(this.playingSecNo >= this.sections.length) {
        this.$timer.stop('running')
        this.state = st.ENDING
      } else {
        // 没有命中的status设为“MISSED”
        if(this.playingSecNo >= 0) {
          let playingSec = this.sections[this.playingSecNo]
          for(let i=this.posInPlayingSec; i<playingSec.length; i++) {
            playingSec[i].status = 'MISSED'
          }
        }
        this.playingSecNo++
        this.posInPlayingSec = 0
      }
    },
    createSections({sectionNum, notesInSection, fingerCount}) {
      this.sections = []
      for(let i=0; i<sectionNum; i++) {
        let section = []
        for(let j=0; j<notesInSection; j++) {
          let finger = Math.floor(Math.random() * fingerCount) + 1
          let status = 'NOT_HIT'
          section.push({finger, status})
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
<style>
  .note {
    display: inline-block;
    width: 1em;
  }
  .succ {
    color: green;
  }
  .fail{
    color: red;
  }
  .home{
    text-align: left;
  }
  .looking-ahead {
    color: blue;
  }
</style>
