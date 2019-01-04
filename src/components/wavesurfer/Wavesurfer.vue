<template>
    <div class="mb-2">
        <div :id="uniqueId"  :class="audioWaveClass" @click="audioState('play')" @mousemove="audioInteraction" :title="audioTime"></div> 

        <div id="bufferContainer">
            <div id="bufferProgress" :style="progressBuffer"></div>
            <div id="currentProgress" :style="loadingBarStyle"></div>

        </div>
        <b-row  class="bg-dark pb-1 pt-1 margin-row" >
            <div class="bg-light loading-bar" :style="loadingBarStyle"></div>
            <b-col>
                <div :class="{'disabled': isError || !audio.recordingUrl }">
                    <a @click="audioState('play')"    :class="audioControlsClass(currentState,'play')">   <i class="fa fa-play hover cursor-pointer"></i>    </a>
                    <a @click="audioState('pause')"   :class="audioControlsClass(currentState,'pause')">  <i class="fa fa-pause hover cursor-pointer"></i>   </a>
                    <a @click="audioState('stop')"    :class="audioControlsClass(currentState,'stop')">   <i class="fa fa-stop hover cursor-pointer"></i>    </a>
                    <i v-if="isNotReady" class="fa fa-spinner fa-pulse text-white"></i>
                </div>
            </b-col>
            <b-col>
                <div class="float-right text-white ">
                    {{timeTracker(currentTime,duration)}}
                </div>
            </b-col>
        </b-row>
        
    </div>
</template>
<script>

/**
 * Wavesurfer Component v1.1
 * 
 * Newly added feature:
 * Buffering indicator
 * 
 * Lacking:
 * Buffering on pause
 */
import wavesurfer from 'wavesurfer.js';
// import handleResponse from '../../../utils/http-response-handler/response-handler';
import { eventBus } from './eventBus';

let componentId = 1;

function getComponentId(){
    return `__WAVE_SURFER_${componentId++}__`;
}

const floor         = Math.floor;
const zeroPrepend   = (int)     => int > 9 ? `${int}` : `0${int}`;
const defaultTime   = (time)    => time || "00";
const returnValue   = (value)   => value  > 0 ? zeroPrepend(value)  : "";
const setTimeFormat = (hour,min,sec,defaultFormat) => (hour || min || sec) ? defaultTime(hour) + ':' + defaultTime(min) + ':' + defaultTime(sec) : defaultFormat;
const minute        = (time) => floor( time % 3600 / 60 );
const second        = (time) => floor( time % 3600 % 60 );
const hour          = (time) => floor( time / 3600 );

export default {
    props:{
        audio: {
            type: Object
        },
    },
    data(){
        return {
            uniqueId            : getComponentId(),
            ws                  : {},
            duration            : 0,
            currentTime         : 0,
            remainingTime       : 0,
            audioTime           : 0,
            currentProgress    : 0,
            bufferProgress      : 0,
            currentState        : null,
            maxWidth            : null,
            isFinishLoading     : false,
            isError             : false,
            isNotReady          : false,
        }
    },
    methods: {
        timeFormatter(time) {
            let defaultFormat = "00:00:00";
            let secondFormat  = returnValue(second(time));
            let minuteFormat  = returnValue(minute(time));
            let hourFormat    = returnValue(hour(time));

            return setTimeFormat(hourFormat,minuteFormat,secondFormat,defaultFormat);
        },
        timeTracker(leftTime,rightTime) {
            return this.timeFormatter(leftTime) + ' / ' + this.timeFormatter(rightTime);
        },
        audioState(audioState) {
            switch(audioState){
                case 'play':
                    this.currentState = audioState;
                    eventBus.$emit('audioPlay', { source: this, recordingId: this.audio.recordingId || null });
                    this.ws.play();
                    break;
                case 'pause':
                    this.currentState = audioState;
                    this.ws.pause();
                    break;
                case 'stop':
                    this.currentState      = audioState;
                    this.currentTime       = 0;
                    this.remainingTime     = this.duration;
                    this.currentProgress   = 0;
                    this.bufferProgress    = 0;
                    this.ws.stop();
                    break;
            }

        },
        audioInitialization(url) {
            this.ws = wavesurfer.create({
                container: '#' + this.uniqueId,
                waveColor: '#A8DBA8',
                progressColor: '#3B8686',
                backend: 'MediaElement',
                mediaType:'audio',
                normalize: true,
                autoCenter: true,
                height: 65,
              
            });
            this.isError    = true;
            this.isNotReady = true;
        },
        audioChanged(url){
                this.currentTime = 0 ;
                this.remainingTime = this.duration;
                this.audioDestroy();
                this.audioInitialization();
                this.audioReady(this,url);
        },
        audioReady(self,url) {
            this.ws.load(url);
            this.ws.on('ready', () => {
                self.isError            = false;
                self.isNotReady         = false;
                self.duration           = floor(self.ws.getDuration());
                self.remainingTime      = self.duration;
                self.currentTime        = 0;
                self.maxWidth           = self.ws.drawer.width
                self.audioState(self.defaultAudioState);
            });
        },
        audioProcessing(self){
            
            this.ws.on('audioprocess', (e) => {
                if ( self.ws.isPlaying() ) {
                    self.currentProgress = self.ws.container.children[0].children[0].clientWidth;
                    self.currentTime     = floor(self.ws.getCurrentTime());
                    self.remainingTime   = self.duration - self.currentTime;
                }

                this.audioBuffered(self.currentTime);
            });

        },
        audioBuffered(currentTime){
            let bufferedLength = this.ws.backend.media.buffered.length;
            let buffered = this.ws.backend.media.buffered;
            if ( this.duration > 0 ) {
                for ( let i = 0 ; i < bufferedLength; i++ ){
                    if (buffered.start(bufferedLength - 1 - i) < currentTime) {
                        let percentage = floor((buffered.end(bufferedLength - 1 - i) / this.duration) * 100);
                        this.bufferProgress = percentage;
                        break;
                    }
                }
            }
        },
        audioSeek(self){
            this.ws.on('seek', (position) => {
                let converted       = floor(position * self.duration);
                self.currentTime    = converted;
                self.remainingTime  = self.duration - self.currentTime;
            });
        },
        audioFinished(self){
            this.ws.on('finish', () => {
                self.audioState('stop');
            });
        },
        audioLoading(){
            let self = this;
            
            this.ws.on('loading', function(time = 0,xhr){
                self.isFinishLoading = (time === 100) ? true : false;
            });
        },
        audioStop(e){
            if ( e.source != this ) {
                return this.audioState('stop');
            }
        },
        audioError(self){
            this.ws.on('error', (error) => { 
                self.isError    = true;
                self.isNotReady = false;
                eventBus.$off('audioPlay', this.audioStop);
                // handleResponse.onError("Can't retrieve audio recording."); 
            });
        },
        audioDestroy(){
            this.ws.empty();
            this.ws.destroy();
        },
        audioInteraction(event){
            this.audioTime = this.timeFormatter((event.layerX / this.ws.drawer.width) * this.ws.getDuration());
        },
        audioControlsClass(currentState,state){
            return {
                'text-white':true,
                'mr-2':true, 
                'text-secondary':currentState === state, 
                'disabled': (currentState === 'stop' && state === 'pause')
            }
        },
        setDefaults(){
            this.ws                  = {};
            this.duration            = 0;
            this.currentTime         = 0;
            this.remainingTime       = 0;
            this.audioTime           = 0;
            this.currentProgress     = 0;
            this.currentState        = null;
            this.maxWidth            = null;
            this.isFinishLoading     = false;
            this.isError             = false;
            this.isNotReady          = false;
        },
    },
    computed: {
        defaultAudioState(){
            return this.audio.audioState ?  this.audio.audioState : 'stop';
        },
        audioWaveClass(){
            return {
                'bg-light':true,
                'cursor-pointer':true,
                'disabled': this.isError || !this.audio.recordingUrl
            }
        },
        progressBuffer(){
            return {
                'width': this.bufferProgress + '%',
                'max-width': this.maxWidth + 'px'
            }
        },
        loadingBarStyle(){
            return {
                'width': this.currentProgress + 'px',
            }
        },
    },
    watch:{
        audio(nv){
            this.audioChanged(nv.recordingUrl);
            this.audioProcessing(this);
            this.audioSeek(this);
        },
    },
    created(){
        this.setDefaults();
    },
    mounted(){
            this.audioInitialization();
            this.audioReady(this,this.audio.recordingUrl);
            this.audioProcessing(this);
            this.audioSeek(this);
            this.audioFinished(this);
            this.audioError(this);
            this.audioLoading(this);
            eventBus.$on('audioPlay', this.audioStop);
    },
    destroyed(){
            this.audioDestroy();
            eventBus.$off('audioPlay', this.audioStop);
            this.setDefaults();
    }
}

</script>

<style scoped>
    .margin{
      margin: 5px 10px 0px 0px;
    }
    .margin-row{
        margin-right: 0;
        margin-left: 0;
    }
    .hover:hover{
        transition: 0.3s ease-in-out;
        color: #6c757d;
    }
    .disabled{
        pointer-events: none;
    }
    
    .loading-bar{
        position:absolute;
        padding-top: 25px;
        padding-bottom: 7px;
        margin-top: -4px;
        opacity:0.05;
    }

    #bufferContainer{
        width: 100%;
        height: 3px;
        background-color: #dcdcdd;
    }

    #currentProgress{
        position: absolute;
        background-color: #ff0000;
        height: inherit;
    }

    #bufferProgress{
        position: absolute;
        background-color: #000000;
        height:inherit;
        opacity: 0.2;
    }
</style>
