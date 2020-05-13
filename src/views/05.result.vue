<template>
  <div class="result-container">
    <div class="title-wrap">
      <h2 class="title" :keywords="(keywords = $route.query.q)">{{ $route.query.q }}</h2>
      <!-- <h2 class="title">{{ keywords }}</h2> -->
      <span class="sub-title">{{ count }}</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item, index) in songs" :key="index" @click="playMusic(item.id)">
              <td>{{ index + 1 }}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <!-- 名称 -->
                    <span>{{ item.name }}</span>
                    <!-- mv图标 -->
                    <span v-if="item.mvid !== 0" class="iconfont icon-mv"></span>
                  </div>
                  <!-- 二级标题 -->
                  <span v-if="item.alias.length !== 0">{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>

      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item, index) in playlists" :key="index" @click="toPlaylist(item.id)">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>

      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item, index) in mvs" :key="index" @click="toMv(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>

import axios from 'axios'

export default { 
  data() {
    return {
      // 总条数
      total: 0,
      // 页码
      pageNum: 1,
      pageSize: 10,
      keywords: '',
      activeIndex: 'songs',
      songs: [],
      count: 0,
      playlists: [],
      mvs: [],
      songId: 0,
      type: 1
    }
  },
  created() {
    axios({
      url: 'https://autumnfish.cn/search',
      method: 'get',
      params: {
        keywords: this.$route.query.q,
        type: 1,
        limit: 10,
      }
    }).then(res => {
      this.songs = res.data.result.songs
      for (let i = 0; i < this.songs.length; i++) {
        let min = parseInt(this.songs[i].duration / 1000 / 60)
        let sec = parseInt((this.songs[i].duration / 1000) % 60)
        if (min < 10) {
          min = '0' + min
        }
        if (sec < 10) {
          sec = '0' + sec
        }
        this.songs[i].duration = min + ':' +sec
      }
      this.count = res.data.result.songCount
    })
  },
  watch: {
    activeIndex() {
      let type = 1
      let limit = 10
      switch (this.activeIndex) {
        case 'songs':
          type = 1
          limit = 10
          break;
        case 'lists':
          type = 1000
          limit = 10
          break;
        case 'mv':
          type = 1004
          limit = 8
          break;  
        default:
          break;
      }
      axios({
        url: 'https://autumnfish.cn/search',
        method: 'get',
        params: {
          keywords: this.$route.query.q,
          type,
          limit,
        }
      }).then(res => {
        // 歌曲
        if (type == 1) {
          this.songs = res.data.result.songs
          for (let i = 0; i < this.songs.length; i++) {
            let min = parseInt(this.songs[i].duration / 1000 / 60)
            let sec = parseInt((this.songs[i].duration / 1000) % 60)
            if (min < 10) {
              min = '0' + min
            }
            if (sec < 10) {
              sec = '0' + sec
            }
            this.songs[i].duration = min + ':' +sec
          }
          this.count = res.data.result.songCount
        }
        // 歌单
        else if (type == 1000) {
          this.playlists = res.data.result.playlists
          this.count = res.data.result.playCount
          for (let i = 0; i < this.playlists.length; i++) {
            if (this.playlists[i].playCount > 100000) {
              this.playlists[i].playCount =  parseInt(this.playlists[i].playCount / 10000 + '万')
            }
            
          }
        }
        // MV
        else {
          this.mvs = res.data.result.mvs
          this.count = res.data.result.mvCount
          // 时间
          for (let i = 0; i < this.mvs.length; i++) {
            let min = parseInt(this.mvs[i].duration / 1000 /60)
            let sec = (parseInt(this.mvs[i].duration / 1000) % 60)
            if (min < 10) {
              min = '0' + min
            }
            if (sec < 10) {
              sec = '0' + sec
            }
            this.mvs[i].duration = min + ':' + sec
            // 次数
            if (this.mvs[i].playCount> 10000) {
              this.mvs[i].playCount = parseInt(this.mvs[i].playCount / 10000) + '万'
            }
          }
        }
      })
    }
  },
  methods: {
    //跳转歌单详情页
    toPlaylist(id) {
      this.$router.push('/playlist?q=' + id)
    },
    playMusic(id) {
      axios({
        url: 'https://autumnfish.cn/song/url',
        method: 'get',
        params: {
          id
        }
      }).then(res => {
        let url = res.data.data[0].url
        // 设置给父组件的播放地址
        this.$parent.musicUrl = url
      })
    },
    // 去MV
    toMv(id) {
      this.$router.push('/mv?q=' + id)
    },
  }
}
</script>

<style >
</style>
