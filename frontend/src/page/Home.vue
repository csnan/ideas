<template>
  <div class="home">
    <base-header :rightIcon="searchIcon" @toPage="toSearch"></base-header>
    <div class="tab-box">
      <div
       class="tab-item"
       v-for="(tab, index) in tabs"
       :key="index"
       @click="onTabsSwitch(index)"
       v-bind:class="{active:index==nowTabIndex}"
      >{{tab}}</div>
    </div>
    <div class="main-content">
      <van-notice-bar
        class="notice-bar"
        text="用户OKOer精通玩家最近斩获IDEAS原创大赛特等奖，超牛逼的！！！"
        left-icon="volume-o"
        mode="closeable"
      />
      <div class="tab-first" v-show="nowTabIndex===0">
        <home-recommend></home-recommend>
      </div>
      <div class="tab-second" v-show="nowTabIndex===1">
        <home-focus></home-focus>
      </div>
    </div>
  </div>
</template>

<script>
import homeRecommend from './homePage/HomeRecommend'
import homeFocus from './homePage/HomeFocus'
export default {
  name: 'home',
  components: {
    homeRecommend, 
    homeFocus
  },
  data() {
    return {
      searchIcon: require('@/assets/images/search.png'),
      nowTabIndex: 0,
      tabs: ["推荐","关注"]
    }
  },
  methods: {
    onTabsSwitch(index) {
      this.nowTabIndex = index
    },
    toSearch() {
      this.$router.push('/search')
    },
  }
}
</script>

<style lang="less" scoped>
.home {
  .tab-box {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    margin: 0 auto;
    width: 50%;
    height: 45px;
    z-index: 999;
    .tab-item{
      width: 50%;
      height: 43px;
      line-height: 50px;
      display: inline-block;
      text-align: center;
      font-size: 14px;
      color: rgb(173, 218, 255);
    }
    .tab-item.active {
      width: 50%;
      height: 43px;
      line-height: 50px;
      display: inline-block;
      text-align: center;
      border-bottom: 2px solid rgb(255, 255, 255);
      font-size: 14px;
      color: #FFF;
    }
  }
  .main-content {
    .notice-bar {
      height: 30px;
      font-size: 12px;
    }
  }
}
</style>
