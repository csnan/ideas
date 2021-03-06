<template>
  <div class="musicPage">
    <loading-image :loadingShow="loadingShow"></loading-image>
    <right-dialog 
      v-show="showDialog" 
      :collectionShow="collectionShow" 
      @collection="onCollection"
      @share="onShare"
      @report="onReport"
    ></right-dialog>
    <base-header :leftLogo="false" :leftIcon="backIcon" :rightIcon="moreIcon" @goBack="toBack" @toPage="onOpenDialog"></base-header>
    <div class="main-content" @click="onCloseDialog">
      <div class="music-wrap">
        <img class="bg-cover" :src="music.idea_img">
        <div class="operate-button" @click="onChangeStatus">
          <img :src="operateIcon">
        </div>
        <van-row class="music-foot">
          <van-col span="4">
            <span>{{time}}</span>
          </van-col>
          <van-col span="16">
            <!-- <div class="dialogAudioProgress" ref="dialogAudioProgress" @mousedown="controlAudioProgress($event)">
              <span class="progressDot" :style="dotStyle"></span>
              <span class="bar" :style="progressStyle"></span>
            </div> -->
            <div @touchstart.prevent="progressTouchStart" @touchmove.prevent="progressTouchMove" @touchend="progressTouchEnd">
              <van-slider v-model="value" bar-height="4px" @change="controlAudioProgress" />
            </div>
          </van-col>
          <van-col class="right-time" span="4">
            <span>{{duration}}</span>
          </van-col>
        </van-row>
        <audio 
          ref="recordAudio"  
          type="audio/mp3" 
          :src="music.idea_file"
          @canplay="canPlay" 
          @timeupdate="timeUpdate" 
          @ended="onEnded"
        >
        </audio>
      </div>
      <div class="music-title">
        {{music.idea_title}}
        <van-icon :class="[rotate?'down-icon-rotate':'down-icon']" name="arrow-down" color="gray" @click="onRotate"/>
      </div>
      <van-cell class="music-author" :title="author.username" :border="false" center>
        <div class="head-image" slot="icon"  @click="toUserCenter">
          <img :src="author.headImg">
        </div>
        <van-button 
          class="focus-button" 
          v-show="showFocus"
          slot="right-icon" 
          size="small" 
          type="primary"
          @click="onFocus()" 
          plain
        >+ 关注</van-button>
      </van-cell>
      <div class="music-detail">
        <span>{{readNum}}次播放</span>
        <span>{{music.idea_time}}</span>
        <transition name="van-fade">
          <div v-show="visible">{{music.idea_content}}</div>
        </transition>
      </div>
      <van-row class="music-four-handle">
        <van-col span="8">
          <div @click="addLikeNum">
            <img :src="likeStatus">
          </div>{{likeNum}}
        </van-col>
        <!-- <van-col span="6">
          <div>
            <img src="../../assets/images/collection2.png">
          </div>5645
        </van-col> -->
        <van-col span="8">
          <div>
            <img src="../../assets/images/comment3.png">
          </div>{{commentNum}}
        </van-col>
        <van-col span="8">
          <div>
            <img src="../../assets/images/share2.png">
          </div>
        </van-col>
      </van-row>
      <div class="gray-block"></div>
      <comment-area :idea_id="idea_id" @commentNum="getCommentNum"></comment-area>
    </div>
  </div>
</template>

<script>
import { postFindOneIdea } from "@/api/index"
import { postAddFocus } from "@/api/index"
import { postObtainUserInfo } from "@/api/index"
import { postUpdateLikeNum } from "@/api/index"
import { postAddLikeUser } from "@/api/index"
import { postReduceLikeUser } from "@/api/index"
import { postAddCollection } from "@/api/index"
import { postUpdateReadNum } from "@/api/index"
import { postAddFootprint } from "@/api/index"
export default {
  name: 'musicPage',
  data() {
    return {
      backIcon: require('@/assets/images/back2.png'),
      moreIcon: require('@/assets/images/more.png'),
      collectionShow: true,
      showDialog: false,
      loadingShow: false,
      showFocus: true,
      operateIcon: require('@/assets/images/play3.png'),
      time: "00:00",
      duration: "00:00",
      // progressStyle: { width: "" },
      // dotStyle: { left: "" },
      value: 0,
      initiated: false,
      rotate: false,
      visible: false,
      music: {},
      idea_id: this.$route.query.music_id,
      author_id: '',
      author: [],
      commentNum: 0,
      likeStatus: require('@/assets/images/like4.png'),
      likeNum: 0,
      readNum: 0
    }
  },
  mounted() {
    this.getMusicDetail()
  },
  methods: {
    toBack() {
      let recordAudio = this.$refs.recordAudio; //获取audio元素
      recordAudio.pause()
      this.$router.go(-1)
    },

    onOpenDialog() {
      this.showDialog = !this.showDialog
    },
    onCloseDialog() {
      this.showDialog = false
    },

    getMusicDetail() {
      this.loadingShow = true
      postFindOneIdea({
        _id: this.$route.query.music_id
      }).then(res => {
        if(res.success) {
          this.loadingShow = false
          this.music = res.resultList
          this.author_id = res.resultList.author_id
          this.likeNum = this.music.like_num
          this.readNum = this.music.read_num
          this.readNum++

          //作者信息
          postObtainUserInfo({
            _id: this.author_id
          }).then(res => {
            this.author = res.resultList
          })

          //访问量
          postUpdateReadNum({
            idea_id: this.$route.query.music_id,
            read_num: this.readNum
          }).then(res => {
            if(res.success) {
            }
          })

          //足迹
          postAddFootprint({
            user_id: this.$store.state.idData,
            footprint_id: this.$route.query.music_id
          }).then(res => {
            if(res.success) {
              this.$store.state.footprintData = res.resultList.footprint_id
            }
          })

          //当作者为用户本身或者已经关注该作者或未登录则隐藏关注按钮
          if(this.author_id == this.$store.state.idData || !this.$store.state.memberData) {
            this.showFocus = false
          }
          for(let i = 0; i < this.$store.state.focusData.length; i++) {
            if(this.author_id == this.$store.state.focusData[i]) {
              this.showFocus = false
            }
          }

          //当作者为用户本身或者已经收藏该作者或未登录则隐藏收藏按钮
          if(this.author_id == this.$store.state.idData || !this.$store.state.memberData) {
            this.collectionShow = false
          }
          for(let i = 0; i < this.$store.state.collectionData.length; i++) {
            if(this.$route.query.music_id == this.$store.state.collectionData[i]) {
              this.collectionShow = false
            }
          }

          //点赞按钮状态
          for(let i = 0; i < this.music.like_user.length; i++) {
            if(this.$store.state.idData == this.music.like_user[i]) {
              this.likeStatus = require('@/assets/images/like44.png')
            }
          }
        }
      })
    },

    toUserCenter() {
      this.$router.push({
        name: 'userCenter',
        query: {
          author_id: this.author_id
        }
      })
    },

    onFocus() {
      postAddFocus({
        _id: this.$store.state.idData,
        focus_id: this.author_id
      }).then(res => {
        if(res.success) {
          this.$store.state.focusData = res.resultList.focus_id
          this.$store.state.focusNumData = res.resultList.focus_id.length
          this.showFocus = false
          this.$toast.success('已关注该作者')
        }
      })
    },

    addLikeNum() {
      if(this.likeStatus == require('@/assets/images/like4.png')) {
        this.likeNum++
        this.likeStatus = require('@/assets/images/like44.png')
        postAddLikeUser({
          idea_id: this.$route.query.music_id,
          like_user: this.$store.state.idData
        }).then(res => {
          if(res.success) {
          }
        })
      } else {
        this.likeNum--
        this.likeStatus = require('@/assets/images/like4.png')
        postReduceLikeUser({
          idea_id: this.$route.query.music_id,
          like_user: this.$store.state.idData
        }).then(res => {
          if(res.success) {
          }
        })
      }

      postUpdateLikeNum({
        idea_id: this.$route.query.music_id,
        like_num: this.likeNum
      }).then(res => {
        if(res.success) {
        }
      })

    },

    onCollection() {
      this.showDialog = false
      postAddCollection({
        user_id: this.$store.state.idData,
        collection_id: this.$route.query.music_id
      }).then(res => {
        if(res.success) {
          this.$store.state.collectionData = res.resultList.collection_id
          this.collectionShow = false
          this.$toast('已收藏此作品')
        }
      })
    },

    onShare() {
      this.showDialog = false
      this.$toast('暂未开放')
    },
    onReport() {
      this.showDialog = false
      this.$router.push({
        name: 'reportPage',
        query: {
          idea_id: this.$route.query.music_id,
          idea_title: this.music.idea_title
        }
      })
    },

    getCommentNum(data) {
      this.commentNum = data
    },
    
    //播放暂停控制
    onChangeStatus() {
      let recordAudio = this.$refs.recordAudio; //获取audio元素
      if (recordAudio.paused) {
        this.operateIcon = require('@/assets/images/pause.png'),
        recordAudio.play();
      } else {
        this.operateIcon = require('@/assets/images/play3.png')
        recordAudio.pause();
      }
    },

    //进度条更新
    timeUpdate() {
      this.duration = this.transTime(this.$refs.recordAudio.duration);
      let timeStr = parseInt(this.$refs.recordAudio.currentTime);
      this.time = this.transTime(timeStr);
      let scales = this.$refs.recordAudio.currentTime / this.$refs.recordAudio.duration;
      if(!this.initiated){
        this.value = scales * 100
      }
      // this.progressStyle.width = scales * 100 + '%';
      // this.dotStyle.left = scales * 100 + '%';
    },

    //播放结束
    onEnded() {
      this.operateIcon = require('@/assets/images/play3.png')
      this.time = "00:00"
      this.value = 0
      // this.progressStyle.width = 0
      // this.dotStyle.left = 0
    },

    //用户可以开始播放audio
    canPlay() {
      //获取audio音频文件总时长 并设置到界面并解决出现 NAN 的问题
      this.duration = this.transTime(this.$refs.recordAudio.duration);
    },

    //点击移动播放进度
    // controlAudioProgress(event) {
    //   let audio = this.$refs.recordAudio;
    //   let dialogAudioProgress = this.$refs.dialogAudioProgress;
    //   if (!audio.paused || audio.currentTime != 0) {
    //     let pgsWidth = parseFloat(window.getComputedStyle(dialogAudioProgress, null).width.replace('px', ''));
    //     let rate = event.offsetX / pgsWidth;
    //     audio.currentTime = audio.duration * rate;
    //     this.timeUpdate();
    //   }
    // },

    //改变播放进度
    controlAudioProgress(value) {
      let audio = this.$refs.recordAudio
      if (!audio.paused || audio.currentTime != 0) {
        audio.currentTime = value * 2
        this.timeUpdate()
      }
    },

    //触摸、移动、结束改变相应状态
    progressTouchStart() {
      this.initiated = true
    },
    progressTouchMove() {
      this.initiated = true
    },
    progressTouchEnd() {
      this.initiated = false
    },

    //时间转换
    transTime(value) {
      let time = "";
      let h = parseInt(value / 3600);
      value %= 3600;
      let m = parseInt(value / 60);
      let s = parseInt(value % 60);
      if (h > 0) {
        time = this.formatTime(h + ":" + m + ":" + s);
      } else {
        time = this.formatTime(m + ":" + s);
      }
      return time;
    },

    //时间格式化
    formatTime(value) {
      let time = "";
      let s = value.split(':');
      let i = 0;
      for (; i < s.length - 1; i++) {
        time += s[i].length == 1 ? ("0" + s[i]) : s[i];
        time += ":";
      }
      time += s[i].length == 1 ? ("0" + s[i]) : s[i];
      return time;
    },

    //实现点击旋转图标180度以及出现简介内容
    onRotate() {
      this.rotate = !this.rotate
      this.visible = !this.visible
    }
  }
}
</script>

<style lang="less" scoped>
.musicPage {
  .main-content {
    .music-wrap {
      width: 100%;
      height: 200px;
      position: relative;
      overflow: hidden;
      background: rgb(0, 0, 0);
      .bg-cover {
        width: 100%;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
      .operate-button {
        width: 30px;
        height: 30px;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        img {
          width: 100%;
          height: 100%;
        }
      }
      .music-foot {
        width: 90%;
        position: absolute;
        left: 0;
        right: 0;
        margin: auto;
        color: #FFF;
        font-size: 12px;
        bottom: 10px;
        display: flex;
        align-items: center;
        // .dialogAudioProgress {
        //   height: 2px;
        //   background: rgba(212, 249, 232, 1);
        //   border-radius: 1px;
        //   position: relative;
        //   .progressDot {
        //     width: 8px;
        //     height: 8px;
        //     border-radius: 50%;
        //     -moz-border-radius: 50%;
        //     -webkit-border-radius: 50%;
        //     background-color: rgba(5, 180, 147, 1);
        //     position: absolute;
        //     left: 0;
        //     top: 50%;
        //     margin-top: -5px;
        //     margin-left: -5px;
        //     cursor: pointer;
        //   }
        //   .bar {
        //     height: 100%;
        //     background: rgba(5, 180, 147, 1);
        //     border-radius: 3px;
        //     display: inline-block;
        //     position: absolute;
        //   }
        // }
        .right-time {
          text-align: right;
        }
      }
    }
    .music-title {
      margin-top: 20px;
      padding: 0 15px;
      font-size: 18px;
      .down-icon {
        vertical-align: top;
        float: right;
        transition: all 0.5s;
      }
      .down-icon-rotate {
        vertical-align: top;
        float: right;
        transform:rotate(-180deg);
        transition: all 0.5s;
      }
    }
    .music-author {
      .head-image {
        width: 35px;
        height: 35px;
        position: relative;
        margin-right: 10px;
        border-radius: 50px;
        overflow: hidden;
        img {
          width: 120%;
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
        }
      }
      .focus-button {
        border: 1px solid rgb(246, 131, 12);
        color: rgb(246, 131, 12);
      }
    }
    .music-detail {
      padding: 0 15px;
      font-size: 12px;
      color: rgb(136, 136, 136);
      span {
        margin-right: 10px;
      }
      div {
        margin-top: 5px;
      }
    }
    .music-four-handle {
      width: 90%;
      margin: 25px auto 10px auto;
      text-align: center;
      font-size: 12px;
      img {
        width: 20px;
        height: 20px;
      }
    }
    .gray-block {
      height: 1px;
      background: rgb(240, 240, 240);
    }
  }
}
</style>
