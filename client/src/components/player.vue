<template>
    <div>
        <div id="morePanel">
            <div class="closeArea" v-on:click="viewMore">
                <ChevronDownIcon id="closeMoreIcon"/>
                <p id="closeMoreTitle">Close</p>
            </div>
            <div class="infoArea">
                <img v-if="ytInfo.snippet.thumbnails.medium.url == 'none'" class="moreImg" src="../assets/medium-img.png" alt="">
                <img v-if="ytInfo.snippet.thumbnails.medium.url != 'none'" class="moreImg" v-bind:src="ytInfo.snippet.thumbnails.medium.url">
                <p id="mobileTitle">{{ ytInfo.snippet.title }}</p>
            </div>
            <div class="controlArea">
                <div v-on:click="barSeek" id="mobileSeekbar">
                    <p class="mobileProgress">{{ Math.round(Math.floor(this.progress / 60))+':'+Math.round(this.progress-(Math.floor(this.progress / 60)*60)) }}</p>
                    <p class="mobileDuration">{{ Math.round(Math.floor(this.duration / 60))+':'+Math.round(this.duration-(Math.floor(this.duration / 60)*60)) }}</p>
                    <div v-if="!isCued" id="mobileBar-bg"></div>
                    <div v-if="!isCued" :style="{ width: (progress/duration)*100 + '%' }" id="mobileProgress-bar"></div>
                    <div v-if="isCued" id="mobilePlaceholder-bar"></div>
                </div>
                <ShuffleIcon class="mobileCtrlL" id="mobileShuffle" v-on:click="shuffle"/>
                <SkipBackIcon class="mobileCtrlL" id="mobileBack" v-on:click="skipBack"/>
                <PlayCircleIcon class="mobileCtrlC" style="padding: 0 13px 0 13px" id="mobilePlay" v-if="!isPlaying" v-on:click="playVideo"/>
                <PauseCircleIcon class="mobileCtrlC" style="padding: 0 13px 0 13px" id="mobilePause" v-if="isPlaying" v-on:click="pauseVideo"/>
                <SkipForwardIcon class="mobileCtrlR" id="mobileSeekF" v-on:click="skip"/>
                <RepeatIcon v-if="!onRepeat" v-on:click="repeat" class="mobileCtrlR" id="mobileRepeat"/>
                <RepeatIcon v-if="onRepeat" v-on:click="repeat" class="mobileCtrlR" id="mobileRepeatO"/>
                <div class="mobileModifiers">
                    <VolumeXIcon id="mobileVolumeIcon" v-on:click="volume=0"/>
                    <input v-model="volume" type="range" name="volume" id="mobileVolume">
                    <Volume2Icon id="mobileVolumeIcon2" v-on:click="volume=100"/>
                </div>
            </div>
        </div>
        <div class="player" v-on:click="viewMore">
            <div class="info">
                <ChevronUpIcon id="moreIcon"/>
                <p class="title">{{ ytInfo.snippet.title }}</p>
                <YoutubeIcon style="padding-right: 10px" class="infoBtn"/>
                <PlusIcon v-on:click="addVideo" class="infoBtn"/>
            </div>
            <div class="controls">
                <youtube width=0 height=0 v-bind:video-id="ytInfo.id.videoId" v-bind:player-vars="playerVars" v-on:cued="cued" v-on:playing="playing" v-on:ended="ended" ref="youtube"></youtube>
                <ShuffleIcon class="ctrlIcon" id="shuffle" v-on:click="shuffle"/>
                <SkipBackIcon class="ctrlIcon" id="back" v-on:click="skipBack"/>
                <PlayCircleIcon class="ctrlIcon" style="padding: 0 13px 0 13px" id="play" v-if="!isPlaying" v-on:click="playVideo"/>
                <PauseCircleIcon class="ctrlIcon" style="padding: 0 13px 0 13px" id="pause" v-if="isPlaying" v-on:click="pauseVideo"/>
                <SkipForwardIcon class="ctrlIcon" id="forward" v-on:click="skip"/>
                <RepeatIcon v-if="!onRepeat" class="ctrlIcon" id="repeat" v-on:click="repeat"/>
                <RepeatIcon v-if="onRepeat" class="ctrlIcon" id="repeatO" v-on:click="repeat"/>
                <div v-on:click="barSeek" id="seekbar">
                    <p class="progress">{{ Math.round(Math.floor(this.progress / 60))+':'+Math.round(this.progress-(Math.floor(this.progress / 60)*60)) }}</p>
                    <p class="duration">{{ Math.round(Math.floor(this.duration / 60))+':'+Math.round(this.duration-(Math.floor(this.duration / 60)*60)) }}</p>
                    <div v-if="!isCued" id="bar-bg"></div>
                    <div v-if="!isCued" :style="{ width: (progress/duration)*100 + '%' }" id="progress-bar"></div>
                    <div v-if="isCued" id="placeholder-bar"></div>
                </div>
            </div>
            <div class="modifiers">
                <Volume1Icon id="volumeIcon"/>
                <input v-model="volume" type="range" name="volume" id="volume">
            </div>
        </div>
    </div>
</template>

<script>
import { mapState } from 'vuex';
import { PlayCircleIcon, PauseCircleIcon, SkipBackIcon, SkipForwardIcon, ChevronUpIcon, ChevronDownIcon, Volume1Icon, Volume2Icon, VolumeXIcon, YoutubeIcon, PlusIcon, ShuffleIcon, RepeatIcon } from 'vue-feather-icons';
import axios from 'axios';

export default {
    name: "player",
    components: {
        PlayCircleIcon, PauseCircleIcon, SkipBackIcon, SkipForwardIcon, ChevronUpIcon, ChevronDownIcon, Volume1Icon, Volume2Icon, VolumeXIcon, YoutubeIcon, PlusIcon, ShuffleIcon, RepeatIcon
    },
    data() {
        return {
            playerVars: {
                autoplay: 0,
                controls: 0,
                disablekb: 1
            },
            progress: 0,
            duration: 0.1,
            volume: 100,
            onRepeat: false,
            isPlaying: false,
            isCued: false
        }
    },
    methods: {
        addVideo: async function() {
            var theVid = this.ytInfo;
            theVid.date = new Date;
            const response = await axios(process.env.VUE_APP_SERVER_ADDRESS + '/add', {
                method: "post",
                data: {video: theVid},
                withCredentials: true
            });
            alert("added to list");
        },
        playVideo: function() {
            if(this.ytInfo.id.videoId!='none') {
                this.$refs.youtube.player.playVideo();
                this.isPlaying = true;
            }
        },
        pauseVideo: function() {
            if(this.ytInfo.id.videoId!=='none') {
                this.$refs.youtube.player.pauseVideo();
                this.isPlaying = false;
            }
        },
        stopVideo: function() {
            if(this.ytInfo.id.videoId!='none') {
                this.$refs.youtube.player.stopVideo();
                this.isPlaying = false;
            }
        },
        updateVideo: async function() {
            if(this.ytInfo.id.videoId!='none') {
                this.progress = await this.$refs.youtube.player.getCurrentTime();
                this.duration = await this.$refs.youtube.player.getDuration();
                this.$refs.youtube.player.setVolume(this.volume);
            }
        },
        seekVideo: function(amount) {
            this.$refs.youtube.player.seekTo(amount);
        },
        playing: function() {
            this.isCued = false;
        },
        ended: function() {
            if(this.onRepeat) {
                this.playVideo();
            } else {
                this.pauseVideo();
                this.seekVideo(0);
            }
            this.isPlaying = false;
        },
        cued: function() {
            this.isCued = true;
            this.playVideo();
        },
        barSeek: function(event) {
            if(window.innerWidth <= '500') {
                this.seekVideo(((event.offsetX)/(document.getElementById('mobileSeekbar').offsetWidth))*this.duration);
            } else if(window.innerWidth > '500') {
                this.seekVideo(((event.offsetX)/(document.getElementById('seekbar').offsetWidth))*this.duration);
            }
        },
        repeat: function() {
            if(!this.onRepeat) {
                this.onRepeat = true;
            } else {
                this.onRepeat = false;
            }
        },
        shuffle: function() {

        },
        skip: function() {

        },
        skipBack: function() {
            if(this.progress >= 5) {
                this.seekVideo(0);
            }
        },
        viewMore: function(event) {
            if(window.innerWidth <= '500') {
                var tag = (event.target.tagName);
                if(tag!='svg' && tag!='polygon' && tag!='circle' && tag!='line') {
                    if(document.getElementById('morePanel').style.bottom == '-100%') {
                        document.getElementById('morePanel').style.bottom = '0';
                    } else {
                        document.getElementById('morePanel').style.bottom = '-100%';
                    }
                }
            }
        }
    },
    computed: {
        ...mapState([
            'ytInfo'
        ])
    },
    created() {
        setInterval(this.updateVideo,250);
    }
}
</script>

<style scoped>
.player {
    position: fixed;
    bottom: 0;
    background-color: rgb(35, 35, 35);
    width: 100%;
    padding: 0 0 0 0;
    height: 90px;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    margin: 0 auto;
    box-shadow: 0 0 8px 0 rgba(0, 0, 0, 0.4);
    user-select: text;
}
.info {
    width: 300px;
    min-width: 165px;
    text-align: left;
    padding: 0 0 0 15px;
}
.title {
    font-size: 14px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    padding: 8px 0 0 0;
    line-height: 1.2;
}
#moreIcon {
    display: none;
}
.infoBtn {
    color: rgb(212, 212, 212);
    width: 24px;
    height: 24px;
}
.infoBtn:hover {
    cursor: pointer;
    color: rgb(177, 58, 58);
}
.controls {
    background-color: none;
    width: 600px;
    min-width: 280px;
    text-align: center;
    padding: 13px 20px 0 20px;
}
#play, #pause {
    width: 37px;
    height: 37px;
    color: rgb(226, 226, 226);
    cursor: pointer;
}
#back, #forward {
    width: 28px;
    height: 28px;
    color: rgb(204, 204, 204);
    cursor: pointer;
    position: relative;
    bottom: 4px;
}
#shuffle, #repeat, #repeatO {
    width: 20px;
    height: 20px;
    color: rgb(204, 204, 204);
    cursor: pointer;
    position: relative;
    bottom: 8px;
    padding: 0 20px 0 20px;
}
#repeatO {
    color: rgb(177, 58, 58);
}
#progress-bar {
    background-color: rgb(212, 212, 212);
    height: 7px;
    position: relative;
    top: -7px;
    border-radius: 10px;
}
#bar-bg {
    width: 100%;
    height: 7px;
    background-color: rgb(50, 50, 50);
    border-radius: 10px;
}
#seekbar {
    height: 7px;
    padding: 5px 0 8px 0;
    margin-top: 8px;
    cursor: pointer;
}
#placeholder-bar {
    width: 100%;
    height: 7px;
    background-color: rgb(60, 60, 60);
    border-radius: 10px;
    cursor: progress;
}
.progress {
    padding: 0;
    margin: 0;
    float: left;
    font-size: 10px;
    color: rgb(218, 218, 218);
    position: relative;
    bottom: 15px;
}
.duration {
    padding: 0;
    margin: 0;
    float: right;
    font-size: 10px;
    color: rgb(218, 218, 218);
    position: relative;
    bottom: 15px
}
.modifiers {
    width: 300px;
    min-width: 165px;
    text-align: right;
    padding: 25px 15px 0 0;
}
#volumeIcon {
    color: rgb(212, 212, 212);
    width: 27px;
    height: 27px;
    position: relative;
    top: 8px;
}
#volume {
    width: 130px;
    height: 7px;
    outline: none;
    -webkit-appearance: none;
    background-color: rgb(50, 50, 50);
    border-radius: 20px;
    border: 0;
    cursor: pointer;
}
#volume::-webkit-slider-runnable-track {
    background: rgb(50, 50, 50);
    height: 7px;
    border-radius: 20px;
}
#volume::-webkit-slider-thumb {
    -webkit-appearance: none;
    background-color: rgb(212, 212, 212);
    height: 7px;
    width: 20px;
    border-radius: 20px;
}
#volume::-moz-range-thumb {
    background-color: rgb(212, 212, 212);
    height: 7px;
    width: 20px;
    border-radius: 20px;
    border: none;
}
#volume::-moz-range-progress {
    background-color: rgb(168, 61, 61);
    border-radius: 20px;
    height: 7px;
}
#morePanel {
    display: none;
}

/*mobile */
@media (max-width: 500px) {
    .player {
        flex-direction: column-reverse;
        height: 60px;
        user-select: text;
    }
    .controls {
        width: 100%;
        padding: 0;
        margin: 0 auto -32px auto;
    }
    .ctrlIcon {
        width: 33px;
        height: 33px;
        float:  right;
        padding: 0;
        margin-top: 22px;
    }
    #back, #forward, #shuffle, #repeat, #repeatO {
        display: none;
    }
    #seekbar {
        padding: 0 0 0 0;
        margin: -12px 0 0 0;
        height: 5px;
        cursor: unset;
    }
    #progress-bar {
        border-radius: 0;
        height: 5px;
        top: -5px;
    }
    #bar-bg {
        border-radius: 0;
        height: 5px;
    }
    #placeholder-bar {
        border-radius: 0;
        height: 5px;
    }
    .progress, .duration {
        display: none;
    }
    .info {
        display: block;
        margin-bottom: 10px;
    }
    .infoBtn {
        display: none;
    }
    .title {
        padding: 0;
        margin-bottom: 10px;
        max-width: 250px;
        font-size: 12px;
        margin-left: 30px;
    }
    #moreIcon {
        color: rgb(163, 163, 163);
        display: block;
        margin-bottom: -31px;
        margin-left: -5px;
        pointer-events: none;
        height: 25px;
        width: 25px;
    }
    .modifiers {
        display: none;
    }

    #morePanel {
        display: block;
        width: 100%;
        height: 100%;
        background-color:rgb(35, 35, 35);
        position: fixed;
        bottom: -100%;
        z-index: 10;
        transition: bottom .2s ease-out;
        text-align: center;
        padding: 0;
        user-select: none;
    }
    .moreImg {
        width: 85%;
        display: block;
        margin: auto;
        border-radius: 5px;
    }
    .closeArea {
        height: 50px;
        display: flex;
        flex-direction: row;
        user-select: none;
    }
    .infoArea {
        position: absolute;
        width: 100%;
        top: 15%;
    }
    #mobileTitle {
        font-size: 15px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        line-height: 1.2;
    }
    .controlArea {
        width: 100%;
        position: absolute;
        bottom: 10%;
    }
    #closeMoreIcon {
        color: rgb(163, 163, 163);
        display: block;
        pointer-events: none;
        margin-top: 15px;
        margin-left: 10px;
        height: 25px;
        width: 25px;
    }
    #closeMoreTitle {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        line-height: 1.2;
        font-size: 12px;
        text-align: left;
        margin: 21px 0 0 15px;
        max-width: 250px;
        color: rgb(163, 163, 163);
    }
    .mobileCtrlL {
        color: rgb(226, 226, 226);
        height: 45px;
        width: 45px;
        padding: 0 10px 0 0;
        margin-bottom: -9px;
        cursor: pointer;
    }
    #mobileShuffle, #mobileRepeat, #mobileRepeatO {
        height: 25px;
        width: 25px;
        margin: 0 10px 1px 10px;
    }
    #mobileRepeatO{
        color: rgb(177, 58, 58);
    } 
    .mobileCtrlR {
        color: rgb(226, 226, 226);
        height: 45px;
        width: 45px;
        padding: 0 0 0 10px;
        margin-bottom: -9px;
        cursor: pointer;
    }
    .mobileCtrlC {
        color: rgb(226, 226, 226);
        height: 50px;
        width: 50px;
        margin: 0 -10px 0 -10px;
        margin-bottom: -12px;
        cursor: pointer; 
    }
    .progress2 {
        display: inline-block;
        margin-right: 10px;
        color: rgb(226, 226, 226);
    }
    .duration2 {
        display: inline-block;
        margin-left: 10px;
        color: rgb(226, 226, 226);
    }

    #mobileProgress-bar {
        background-color: rgb(212, 212, 212);
        height: 9px;
        position: relative;
        top: -9px;
        border-radius: 10px;
    }
    #mobileBar-bg {
        width: 100%;
        height: 9px;
        background-color: rgb(50, 50, 50);
        border-radius: 10px;
    }
    #mobileSeekbar {
        height: 9px;
        padding: 5px 0 12% 0;
        cursor: pointer;
        width: 85%;
        margin: auto;
    }
    #mobilePlaceholder-bar {
        width: 100%;
        height: 9px;
        background-color: rgb(60, 60, 60);
        border-radius: 10px;
        cursor: progress;
    }
    .mobileProgress {
        padding: 0;
        margin: 0;
        float: left;
        font-size: 10px;
        color: rgb(218, 218, 218);
        position: relative;
        bottom: 15px;
    }
    .mobileDuration {
        padding: 0;
        margin: 0;
        float: right;
        font-size: 10px;
        color: rgb(218, 218, 218);
        position: relative;
        bottom: 15px
    }

    .mobileModifiers {
        padding-top: 15%;
    }
    #mobileVolumeIcon {
        color: rgb(212, 212, 212);
        width: 27px;
        height: 27px;
        margin-top: -6px;
        cursor: pointer;
    }
    #mobileVolumeIcon2 {
        color: rgb(212, 212, 212);
        width: 27px;
        height: 27px;
        margin-top: -6px;
        cursor: pointer;
    }
    #mobileVolume {
        width: 150px;
        height: 10px;
        outline: none;
        -webkit-appearance: none;
        background-color: rgb(50, 50, 50);
        border-radius: 20px;
        border: 0;
        position: relative;
        bottom: 8px;
        margin: 0 10px 0 10px;
        cursor: pointer;
    }
    #mobileVolume::-webkit-slider-runnable-track {
        background: rgb(50, 50, 50);
        height: 10px;
        border-radius: 20px;
    }
    #mobileVolume::-webkit-slider-thumb {
        -webkit-appearance: none;
        background-color: rgb(212, 212, 212);
        height: 10px;
        width: 20px;
        border-radius: 20px;
    }
    #mobileVolume::-moz-range-thumb {
        background-color: rgb(212, 212, 212);
        height: 10px;
        width: 20px;
        border-radius: 20px;
        border: none;
    }
    #mobileVolume::-moz-range-progress {
        background-color: rgb(168, 61, 61);
        border-radius: 20px;
        height: 10px;
    }
}
</style>