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
        this.channels = response.data.items;
      }).catch((e) =>{
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
        this.playlistId = response.data.items[0].contentDetails.relatedPlaylists.uploads;
        this.getPlaylistItemsList(this.playlistId);
      }).catch((e) =>{
        this.errors = e.response.data.error.message;
      })
    },
    getPlaylistItemsList(playlistId, nextPageToken = null) {
      if (nextPageToken) {
        axios.get(
          YOUTUBE_API_PLAYLISTITEMS_URL + '?part=id,contentDetails,snippet,status' + '&playlistId=' + playlistId + '&maxResults=50' + '&key=' + this.youtubeAPIKey + '&pageToken=' + nextPageToken
        ).then((res) => {
          this.pushItems(res);
        }).catch((e) =>{
        this.errors = e.response.data.error.message;
        })
      } else {
        axios.get(
          YOUTUBE_API_PLAYLISTITEMS_URL + '?part=id,contentDetails,snippet,status' + '&playlistId=' + playlistId + '&maxResults=50' + '&key=' + this.youtubeAPIKey
        ).then((res) => {
          this.pushItems(res);
        }).catch((e) =>{
        this.errors = e.response.data.error.message;
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
        this.reversed = this.playlistItems.reverse();
        this.message = "検索結果：";
      }
    },
    imageClicked(e) {
      let youtube = e.currentTarget.getAttribute('youtube');
      let iframe = '<iframe src="' + youtube + '" frameborder="0"></iframe>';
      e.currentTarget.outerHTML = iframe;
    }
  },
}
</script>

<template>
  <div>
    <label>YouTube API Key: </label>
    <input type='text' v-model="this.youtubeAPIKey">
    <a href="https://www.google.com/search?q=youtube+data+api+key+%E5%8F%96%E5%BE%97" target="_blank" rel="noopener noreferrer" style="text-align: right">APIキー取得方法をググる</a>
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
  <div v-for="item in reversed" :key="item.title">
    <div class="yt">
      <div @click="imageClicked" class="yt_video" v-bind:youtube="'https://www.youtube.com/embed/' + item.videoId + '?rel=0&showinfo=0&autoplay=0'">
		    <img v-bind:src="'https://i.ytimg.com/vi/' + item.videoId + '/mqdefault.jpg'" alt="サンプル動画" width="100%" height="auto" />
        <span>{{ item.title }}</span>
	    </div>
    </div>
  </div>
</template>

<style scoped>
</style>
