<template>
  <div class="playlists-container">
    <!-- 同步 -->
    <div class="top-card">
      <!-- 封面 -->
      <div class="icon-wrap">
        <img :src="topPlayList.coverImgUrl" alt="" />
      </div>
      <div class="content-wrap">
        <div class="tag">精品歌单</div>
        <!-- 标题 -->
        <div class="title">
          {{ topPlayList.title }}
        </div>
        <!-- 介绍 -->
        <div class="info">
          {{ topPlayList.description }}
        </div>
      </div>
      <!-- 背景 -->
      <img :src="topPlayList.coverImgUrl" alt="" class="bg" />
      <div class="bg-mask"></div>
    </div>

    <div class="tab-container">
      <!-- tab栏 顶部 -->
      <div class="tab-bar">
        <!-- active高亮 -->
        <span class="item" @click="tag = '全部'" :class="{ active: tag === '全部' }">全部</span>
        <span class="item" @click="tag = '欧美'" :class="{ active: tag === '欧美' }">欧美</span>
        <span class="item" @click="tag = '华语'" :class="{ active: tag === '华语' }">华语</span>
        <span class="item" @click="tag = '流行'" :class="{ active: tag === '流行' }">流行</span>
        <span class="item" @click="tag = '说唱'" :class="{ active: tag === '说唱' }">说唱</span>
        <span class="item" @click="tag = '摇滚'" :class="{ active: tag === '摇滚' }">摇滚</span>
        <span class="item" @click="tag = '民谣'" :class="{ active: tag === '民谣' }">民谣</span>
        <span class="item" @click="tag = '电子'" :class="{ active: tag === '电子' }">电子</span>
        <span class="item" @click="tag = '轻音乐'" :class="{ active: tag === '轻音乐' }">轻音乐</span>
        <span class="item" @click="tag = '影视原声'" :class="{ active: tag === '影视原声' }">影视原声</span>
        <span class="item" @click="tag = 'ACG'" :class="{ active: tag === 'ACG' }">ACG</span>
        <span class="item" @click="tag = '怀旧'" :class="{ active: tag === '怀旧' }">怀旧</span>
        <span class="item" @click="tag = '治愈'" :class="{ active: tag === '治愈' }">治愈</span>
        <span class="item" @click="tag = '旅行'" :class="{ active: tag === '旅行' }">旅行</span>
      </div>
      <!-- tab的内容区域 -->
      <div class="tab-content">
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
      </div>
    </div>
    <!-- 分页器 -->
    <el-pagination @current-change="handleCurrentChange" background layout="prev, pager, next" :total="total" :current-page="pageNum" :page-size="pageSize">
    </el-pagination>
  </div>
</template>

<script>

import axios from 'axios'

export default {
  name: 'recommend',
  data() {
    return {
      // 总条数
      total: 0,
      // 页码
      pageNum: 1,
      pageSize: 10,
      tag: '全部',
      // 顶部精品歌单
      topPlayList: {},
      // 歌单列表
      playlists: []
    }
  },
  watch: {
    tag () {
      // 顶部精品
      this.topData()
       // 歌单列表
      this.listData()
    }
  },
  created () {
    // 顶部精品
    this.topData()
     // 歌单列表
    this.listData()
    this.pageNum = 1
  },
  methods: {
    topData() {
      // 顶部精品
      axios({
        url: 'https://autumnfish.cn/top/playlist/highquality',
        method: 'get',
        params: {
          limit: 1,
          cat: this.tag
        }
      }).then(res => {
        this.topPlayList = res.data.playlists[0]
      })
    },
    listData() {
      // 歌单列表
      axios({
        url: 'https://autumnfish.cn/top/playlist/',
        method: 'get',
        params: {
          limit: 10,
          // （页码-1）*每页多少条数据
          offset: (this.pageNum -1)*10,
          cat: this.tag
        }
      }).then(res => {
        this.total = res.data.total
        this.playlists = res.data.playlists
      })
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this.listData()
    },
    toPlaylist(id) {
      this.$router.push('/playlist?q=' + id)
    }
  }
}
</script>

<style >
</style>
