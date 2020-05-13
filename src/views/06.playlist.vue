<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <!-- 封面 -->
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <!-- 名字 -->
        <p class="title">{{ playlist.name }}</p>
        <div class="author-wrap">
          <!-- 头像 -->
          <!-- 神经病VUE,不加v-if="playlist.creator !== undefined" 报错 -->
          <img class="avatar" :src="playlist.creator.avatarUrl" v-if="playlist.creator !== undefined" alt="" />
          <!-- 名字 -->
          <span class="name" v-if="playlist.creator !== undefined">{{ playlist.creator.nickname }}</span>
          <!-- 时间 -->
          <span class="time">{{ playlist.createTime }}</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item, index) in playlist.tags" :key="index">{{ item }}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <span class="desc">{{ playlist.description }}</span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item, index) in playlist.tracks" :key="index" @click="playMusic(item.id)">
              <td>{{ index + 1 }}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl+'?param=130y130'" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <!-- 名称 -->
                    <span>{{ item.name }}</span>
                    <!-- mv图标 -->
                    <span v-if="item.mvid !== 0" class="iconfont icon-mv"></span>
                  </div>
                </div>
              </td>
              <td>{{ item.ar[0].name }}</td>
              <td>{{ item.al.name }}</td>
              <td>{{ item.dt }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane :label="`评论(${total})`" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap" v-if="hotComments !== undefined && hotComments.length !== 0">
          <p class="title">
            精彩评论<span class="number">({{ hotComments.length }})</span>
          </p>
          <div class="comments-wrap">
            <!-- 评论 -->
            <div class="item" v-for="(item, index) in hotComments" :key="index">
              <div class="icon-wrap">
                <!-- 头像 -->
                <img :src="item.user.avatarUrl+'?param=50y50'" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{ item.user.nickname }}</span>
                  <span class="comment">：{{ item.content }}</span>
                </div>
                <!-- 回复 -->
                <div class="re-content" v-if="item.beReplied.length !== 0">
                  <span class="name">{{ item.beReplied[0].user.nickname }}</span>
                  <span class="comment">：{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">{{ item.time }}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">
            最新评论<span class="number">({{ total }})</span>
          </p>
          <div class="comments-wrap">
            <!-- 评论 -->
            <div class="item" v-for="(item, index) in comments" :key="index">
              <div class="icon-wrap">
                <!-- 头像 -->
                <img :src="item.user.avatarUrl+'?param=50y50'" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{ item.user.nickname }}</span>
                  <span class="comment">：{{ item.content }}</span>
                </div>
                <!-- 回复 -->
                <div class="re-content" v-if="item.beReplied.length !== 0">
                  <span class="name">{{ item.beReplied[0].user.nickname }}</span>
                  <span class="comment">：{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">{{ item.time }}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 分页器 -->
        <el-pagination @current-change="handleCurrentChange" background layout="prev, pager, next" :total="total" :current-page="pageNum" :page-size="pageSize">
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>

import axios from 'axios'

export default {
  name: 'playlist',
  data() {
    return {
      id: '',
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      pageNum: 1,
      pageSize: 10,
      // 歌单
      playlist: {},
      // 热门评论
      hotComments: [],
      hotCount: 0,
      // 普通评论
      comments: []
    }
  },
  created() {
    // 歌单
    axios({
      url: 'https://autumnfish.cn/playlist/detail',
      method: 'get',
      params: {
        id: this.$route.query.q
      }
    }).then(res => {
      this.playlist = res.data.playlist
    })
    // 评论
    axios({
      url: 'https://autumnfish.cn/comment/hot',
      method: 'get',
      params: {
        id: this.$route.query.q,
        type: 2
      }
    }).then(res => {
      this.hotComments = res.data.hotComments
      this.hotCount = res.data.total
      for (let i = 0; i < this.hotComments.length; i++) {
        let min = parseInt(this.hotComments[i].duration / 1000 / 60)
        let sec = (parseInt(this.hotComments[i].duration / 1000) % 60)
        if (min < 10) {
          min = '0' + min
        }
        if (sec < 10) {
          sec = '0' + sec
        }
        this.hotComments[i].duration = min + ':' + sec
      }
    })
    // 最新评论
    axios({
      url: 'https://autumnfish.cn/comment/playlist',
      method: 'get',
      params: {
        id: this.$route.query.q,
        limit: 10,
        offset: (this.pageNum -1 ) * 10
      }
    }).then(res => {
      this.total = res.data.total
      this.comments = res.data.comments
      for (let i = 0; i < this.comments.length; i++) {
        let min = parseInt(this.comments[i].duration / 1000 / 60)
        let sec = (parseInt(this.comments[i].duration / 1000) % 60)
        if (min < 10) {
          min = '0' + min
        }
        if (sec < 10) {
          sec = '0' + sec
        }
        this.comments[i].duration = min + ':' + sec
      }
    })
  },
  methods: {
    handleCurrentChange(val) {
      this.pageNum = res.data.total
      this.comments = res.data.comments
    },
        // 播放歌曲
    playMusic(id) {
      axios({
        url: 'https://autumnfish.cn/song/url',
        method: 'get',
        params: {
          id
        }
      }).then(res => {
        let url = res.data.data[0].url
        //this.$parent可以直接获取父组件
        this.$parent.musicUrl = url
      })
    }
  }
}
</script>

<style >
</style>
