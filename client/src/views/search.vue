<template>
    <div class="container">
        <div class="results">
            <div class="result" v-on:click="setVideo(searchResults[index])" v-for="(object,index) in searchResults" v-bind:key="index">
                <img class="image" v-bind:src="searchResults[index].snippet.thumbnails.default.url">
                <button class="nBtn">{{ index+1 }}</button>
                <button v-on:click.stop v-on:click="addVideo(searchResults[index])" class="oBtn">+</button>
                <p class="title">{{ searchResults[index].snippet.title }}</p>
            </div>
            <button v-if="searchResults.length > 0" class="loadBtn" v-on:click="loadMore">Load more</button>
        </div>
    </div>
</template>

<script>
import searchYoutube from 'youtube-api-v3-search';
import axios from 'axios';
import { mapMutations } from 'vuex';

export default {
    name: "search",
    components: {
        
    },
    data() {
        return {
            searchTerm: "",
            searchResults: [],
            nextPage: "",
            apiKey: ""
        }
    },
    methods: {
        search: async function() {
            var options = {
                q:this.searchTerm,
                part:'snippet',
                type:'video',
                topicId:'/m/04rlf',
                maxResults: 10
            }
            var results = await searchYoutube(this.apiKey, options);
            this.searchResults = results.items;
            this.nextPage = results.nextPageToken;
        },
        loadMore: async function() {
            var options = {
                q:this.searchTerm,
                part:'snippet',
                type:'video',
                topicId:'/m/04rlf',
                maxResults: 10,
                pageToken: this.nextPage
            }
            var results = await searchYoutube(this.apiKey, options);
            this.nextPage = results.nextPageToken;
            this.searchResults = this.searchResults.concat(results.items);
        },
        setVideo: function(video) {
            this.SET_VIDEO(video);
        },
        addVideo: async function(video) {
            var theVid = video;
            theVid.date = new Date;
            const response = await axios(process.env.VUE_APP_SERVER_ADDRESS + '/add', {
                method: "post",
                data: {video: theVid},
                withCredentials: true
            });
            alert("added to list");
        },
        ...mapMutations([
            'SET_VIDEO'
        ])
    },
    mounted() {
        if(this.$cookie.get('public_key')) {
            this.apiKey = process.env.VUE_APP_APIKEY;
        } else {
            if(this.$cookie.get('api_key') && this.$cookie.get('api_key')!="") {
                this.apiKey = this.$cookie.get('api_key');
            } else {
                alert("You have to add YouTube API key in the settings!");
            }
        }
        this.searchTerm = this.$route.query.q;
        this.search();
    },
    watch: {
        $route() {
            if(this.$cookie.get('api_key') && this.$cookie.get('api_key')!="") {
                this.apiKey = this.$cookie.get('api_key');
            } else {
                alert("You have to add YouTube API key in the settings!");
            }
            this.searchTerm = this.$route.query.q;
            this.search();
        }
    }
}
</script>

<style scoped>
.container {
    /* +180px from left is the sidebar width */
    margin-left: 200px;
    margin-right: 20px;
    /* margins for the navbar and player */
    padding: 80px 0 90px 0;
}
.loadBtn {
    background-color: rgb(168, 61, 61);
    border: none;
    color: white;
    cursor: pointer;
    display: block;
    margin: 20px auto 20px auto;
    padding: 10px 30px 10px 30px;
    border-radius: 20px;
}
.results {
    
}
.result {
    background-color: rgb(20, 20, 20);
    margin: 5px auto 5px auto;
    height: 50px;
    width: 100%;
    display: flex;
    border-radius: 5px;
    align-items: center;
    cursor: pointer;
}
.result:hover {
    background-color: rgb(25, 25, 25);
}
.image {
    height: 100%;
    border-radius: 5px 0 0 5px;
}
.title {
    font-size: 13px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    padding: 0 10px 0 0px;
}
.nBtn {
    background-color: rgb(20, 20, 20);
    border: none;
    color: rgb(196, 196, 196);
    height: 100%;
    min-width: 40px;
    font-size: 13px;
}
.result:hover .nBtn {
    display: none;
}
.result:hover .oBtn {
    display: block;
}
.oBtn {
    background-color: rgb(25, 25, 25);
    border: none;
    color: rgb(196, 196, 196);
    height: 100%;
    min-width: 40px;
    font-size: 18px;
    cursor: pointer;
    display: none;
}
.oBtn:hover {
    color: white;
}
@media (max-width: 860px) {
    .container {
        margin-left: 75px;
        margin-right: 10px;
        padding-top: 70px;
    }
}
@media (max-width: 500px) {
    .nBtn {
        display: none;
    }
    .oBtn {
        display: block;
        background-color: rgb(20, 20, 20);
    }
    .result:hover .oBtn {
        background-color: rgb(25, 25, 25);
    }
}
</style>