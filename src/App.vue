<script>
import axios from 'axios'

const YOUTUBE_API_BASE_URL = 'https://www.googleapis.com/youtube/v3';
const YOUTUBE_API_SEARCH_URL = YOUTUBE_API_BASE_URL + '/search';
const YOUTUBE_API_CHANNELS_URL = YOUTUBE_API_BASE_URL + '/channels';
const YOUTUBE_API_PLAYLISTITEMS_URL = YOUTUBE_API_BASE_URL + '/playlistItems';
let channels = [];
let playlistItems = [];
let reversed = [];
let playlistId = "";

export default {
  data() {
    return {
      youtubeAPIKey: localStorage.getItem('youtubeAPIKey') ? localStorage.getItem('youtubeAPIKey') : '',
      searchWord: '',
      channels: channels,
      playlistItems: playlistItems,
      errors: '',
      message:''
    }
  },
  methods: {
    clear() {
      this.searchWord = '';
    },
    search(searchWord) {
      this.playlistItems = [];
      this.reversed = [];
      this.message = "";
      localStorage.setItem('youtubeAPIKey', this.youtubeAPIKey);
      axios.get(
        YOUTUBE_API_SEARCH_URL + '?part=id&part=snippet' + '&type=channel' + '&order=videoCount' + '&maxResults=5' + '&q=' + searchWord + '&key=' + this.youtubeAPIKey
      ).then((response) => {
        console.log("search: ");
        console.log(response.data.items);
        this.channels = response.data.items;
      }).catch((e) =>{
        console.log(e);
        this.errors = e.response.data.error.message;
      })
    },
    getContents(channel) {
      this.playlistItems = [];
      this.reversed = [];
      this.message = "検索&ソート中";
      axios.get(
        YOUTUBE_API_CHANNELS_URL + '?part=id,contentDetails,snippet,status' + '&id=' + channel.id.channelId + '&key=' + this.youtubeAPIKey
      ).then((response) => {
        console.log("channels: ");
        console.log(response);
        this.playlistId = response.data.items[0].contentDetails.relatedPlaylists.uploads;
        this.getPlaylistItemsList(this.playlistId);
      })
    },
    getPlaylistItemsList(playlistId, nextPageToken = null) {
      if (nextPageToken) {
        axios.get(
          YOUTUBE_API_PLAYLISTITEMS_URL + '?part=id,contentDetails,snippet,status' + '&playlistId=' + playlistId + '&maxResults=50' + '&key=' + this.youtubeAPIKey + '&pageToken=' + nextPageToken
        ).then((res) => {
          console.log(res);
          this.pushItems(res);
        })
      } else {
        axios.get(
          YOUTUBE_API_PLAYLISTITEMS_URL + '?part=id,contentDetails,snippet,status' + '&playlistId=' + playlistId + '&maxResults=50' + '&key=' + this.youtubeAPIKey
        ).then((res) => {
          console.log(res);
          this.pushItems(res);
        })
      }
    },
    pushItems(response) {
      response.data.items.forEach(element => {
        this.playlistItems.push(
          {
            title: element.snippet.title,
            videoId: element.contentDetails.videoId
          }
        );
      });
      if (response.data.nextPageToken) {
        this.getPlaylistItemsList(this.playlistId, response.data.nextPageToken);
      } else {
        console.log("pushItems is completed.");
        this.reversed = this.playlistItems.reverse();
        console.log(this.reversed);
        this.message = "検索結果：";
      }
    },
  },
}
</script>

<template>
  <div>
    <label>YouTube API Key: </label>
    <input type='text' v-model="this.youtubeAPIKey">
  </div>
  <div>
    <label>検索ワード: </label>
    <input type='text' v-model="this.searchWord">
    <button @click="search(this.searchWord)">検索</button>
    <button @click="clear">クリア</button>
  </div>
  <span>{{ errors }}</span>
  <span v-if="channels.length > 0">候補チャンネル(チャンネル名をクリックしてください)</span>
  <ul>
    <li v-for="channel in channels" :key="channel.id" @click="getContents(channel)">
      {{ channel.snippet.title }}
    </li>
  </ul>
  <span>{{ message }}</span>
  <ul>
    <li v-for="item in reversed" :key="item.title">
      <a v-bind:href="'https://youtu.be/' + item.videoId" target="_blank" rel="noopener noreferrer">{{ item.title }}</a>
    </li>
  </ul>
</template>

<style scoped>

</style>
