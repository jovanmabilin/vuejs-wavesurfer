<template>
  <div>
      <header class="bg-dark"> <h1>Wavesurfer Component</h1> </header>
          <!-- <b-container class="mb-5">
            <b-row>
              
              <b-col>
                <b-card class="" header="Audio Track">
                  <b-card-body>
                    <WaveSurfer :audio="selected" class="mb-3"></WaveSurfer>
                    <h3 class="mb-3 mt-5">Audio track</h3>
                    <b-button v-for="(audio,index) in audios" :key="index" variant="primary mr-2" @click="audioChanger(audio)">{{ audio.name }}</b-button>
                  </b-card-body>
                </b-card>
              </b-col>
            </b-row>
          </b-container> -->
        <b-container>
          <b-row>
            <b-col>
              <WaveSurfer v-for="(audio,index) in audios" :key="index" :audio="audio"></WaveSurfer>
            </b-col>
          </b-row>
        </b-container>

  </div>
</template>

<script>
import WaveSurfer from './wavesurfer/Wavesurfer.vue';
import { eventBus } from './wavesurfer/eventBus';
export default {
  components:{
    WaveSurfer,
  },
  data () {
    return {
      selected: {},
      audio1:{
        id: 'audio1',
        name:'Stereo',
        duration: 0,
        recordingUrl: 'https://wavesurfer-js.org/example/split-channels/stereo.mp3',
      },
      audio2:{
        id: 'audio2',
        name: 'The GodFather',
        duration: 0,
        recordingUrl: 'https://ia800508.us.archive.org/15/items/LoveThemeFromTheGodfather/02LoveThemeFromTheGodfather.mp3',
      },
      audio3:{
        id: 'audio3',
        duration: 0,
        recordingUrl: 'https://wavesurfer-js.org/example/elan/transcripts/001z.mp3',
      },

      audios:[
        {
          name: 'Thunderstorm',
          recordingId: 3,
          audioState:'play',
          recordingUrl:'https://ia600700.us.archive.org/25/items/1HourThunderstorm/1HrThunderstorm.mp3',
        },
        {
          name:'Stereo',
          recordingId: 1,
          recordingUrl:'a',

        },
        {
          name: 'The GodFather',
          recordingId: 2,
          recordingUrl:'https://ia800508.us.archive.org/15/items/LoveThemeFromTheGodfather/02LoveThemeFromTheGodfather.mp3',
        },
        
        
      ],
     
    }
  },
  methods:{
    audioChanger(audio, e){
        this.selected = audio;
    },
  },
  watch:{
    selected(nv){
      this.selected = nv;
    }
  },
  created(){
    this.selected = this.audio1;
  },
}
</script>

<style scoped>
  header{
    text-align: center;
    color: #ffffff;
    padding: 20px;
    margin-bottom: 20px;
  }
</style>
