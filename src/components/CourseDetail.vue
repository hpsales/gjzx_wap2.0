<template>
  <div class="coursedetail">
    <div class="backArrow" @click="goback"><img src="../assets/imgs/live_lift@2x.png"></div>
    <videoPlayer :options='playerOptions' ref="videoPlayer" class="video-player vjs-custom-skin" :playsinline="true" @ended="onPlayerEnded()" @play="onPlayerPlay()"></videoPlayer>
    <ul class="courseDetailClass" id="courseDetailClass">
      <li class="activeSelect" @click="selectMenu('introduce')">简介</li>
      <li @click="selectMenu('catalogue')">目录</li>
    </ul>
    <swiper :options="swiperOption" ref="myswiper">
      <swiper-slide>
        <span class="courseDetail_courseName">{{courseName}}</span>
        <p class="courseDetail_courseintro" v-html="intro"></p>
        <p class="courseDetail_coursetotalInfo"><span class="courseDetail_courseClass">课程类别：{{courseClassify}}</span>
        <span class="courseDetail_courseTeacher">主讲老师：{{teacherName}}</span>
        <span class="courseDetail_coursetotalMic">微课总数：{{totalMic}}</span></p>
      </swiper-slide>
      <swiper-slide>
        <ul class="courseknowPoints">
          <li v-for="(item, index) in knowPointsTitleList" :key="index" @click="selectTitle(item.id)">
            <span class="point" :class="{'selectPoint': selectIndex === item.id}"></span>
            <span class="knowPointName" :class="{'select': selectIndex === item.id}">{{item.knowPointName}}</span>
            <img :src="selectIndex === item.id ? courseTitleImg[1] : courseTitleImg[0]">
            <ul class="courseknowPointsVideos" v-show="selectIndex === item.id">
              <li v-for="(item, index) in knowPointsChapterList[index]" :key="index">
                <img src="../assets/imgs/Play@2x.png" class="icon_video">
                <span :class="{'selectChapter': selectChapterIndex === item.id, 'learnedKnowPoint': learnedCourse.includes(item.id)}" @click="selectChapter(item.id)">{{item.knowPointName}}</span>
                <button @click="goTask(item.id, $route.query.courseId)">课堂作业</button>
              </li>
            </ul>
          </li>
        </ul>
      </swiper-slide>
    </swiper>
    <div class="courseDetailfooter clearMargin_top">
      <span>{{price}}</span>
      <span>{{studyCount}}人已学过</span>
      <button @click="handleBuy">{{btnText}}</button>
    </div>
  </div>
</template>

<script>
import 'video.js/dist/video-js.css'
import 'vue-video-player/src/custom-theme.css'
import 'vant/lib/dialog/style'
import {videoPlayer} from 'vue-video-player'
import axios from 'axios'
import Dialog from 'vant/lib/dialog'
export default {
  name: '',
  data () {
    return {
      playerOptions: {
        autoplay: false,
        muted: false,
        loop: false,
        language: 'zh-CN',
        fluid: true,
        aspectRatio: '16:9',
        preload: 'auto',
        sources: [{
          type: 'video/mp4',
          src: 'http://www.gk0101.com/upload/video/65f817ed99414d1ca930a101b413b9a0.mp4'
        }],
        controlBar: {
          timeDivider: false,
          durationDisplay: true,
          remainingTimeDisplay: false,
          fullscreenToggle: true
        }
      },
      swiperOption: {
        on: {
          slideChangeTransitionEnd () {
            let courseDetailClassLi = document.getElementById('courseDetailClass').getElementsByTagName('li')
            for (let i = 0; i < courseDetailClassLi.length; i++) {
              if (i === this.activeIndex) {
                courseDetailClassLi[i].setAttribute('class', 'activeSelect')
              } else {
                courseDetailClassLi[i].setAttribute('class', '')
              }
            }
          }
        }
      },
      totalMic: 0,
      knowPointId: 0,
      courseName: '',
      intro: '',
      teacherName: '',
      courseClassify: '',
      price: '免费',
      studyCount: '',
      firstPlay: true,
      isBuy: false,
      knowPointsTotalList: [],
      knowPointsTitleList: [],
      knowPointsChapterList: [],
      knowPointsTitleListIndex: [],
      selectIndex: 0,
      selectChapterIndex: 0,
      studyData: '',
      learnedCourse: [],
      btnText: '购买课程',
      userID: localStorage.getItem('userID') ? localStorage.getItem('userID') : '',
      courseTitleImg: ['./static/img/direction@2x.png', './static/img/live_right_spread.png']
    }
  },
  components: {
    videoPlayer
  },
  methods: {
    goback () {
      this.$router.go(-1)
    },
    handleBuy () {
      if (!this.userID) return this.$router.push('/mine')
      else {
        if (this.isBuy) {
          // 用户已经购买，控制视频播放
          if (this.$refs.videoPlayer.player.paused) {
            this.$refs.videoPlayer.player.play()
          }
        } else {
          // 用户没有购买，跳转购买课程路由组件
          this.$router.push({
            name: 'CourseBuy',
            query: {
              courseId: this.$route.query.courseId
            }
          })
        }
      }
    },
    goLogin () {
      this.$router.push('/mine')
    },
    goTask (knowPointID, courseID) {
      if (!this.userID) {
        // 跳转登录页
        return this.$router.push('/mine')
      } else {
        if (!this.isBuy) {
          return Dialog.alert({message: '请先购买课程！', confirmButtonColor: '#3246d8', closeOnPopstate: true, closeOnClickOverlay: true})
        } else {
          // 发送Ajax请求判断用户是否已经做过习题
          axios.get(`http://www.gk0101.com/exam/rest/v1/exam/stuPaper?userId=${this.userID}&courseId=${courseID}&examType=0&institutionId=10103&knowPointId=${knowPointID}`).then(res => {
            if (res.data.code === 0) {
              const result = res.data.data
              if (result.view === 'yes') {
                this.$router.push({
                  name: 'CompleteTask',
                  query: {
                    courseId: courseID,
                    knowPointId: knowPointID
                  }
                })
              } else if (result.view === 'no') {
                this.$router.push({
                  name: 'Task',
                  query: {
                    courseId: courseID,
                    knowPointId: knowPointID
                  }
                })
              }
            } else {
              Dialog.alert({message: '习题正在准备中...', confirmButtonColor: '#3246d8', closeOnPopstate: true, closeOnClickOverlay: true})
            }
          })
        }
      }
    },
    selectMenu (str) {
      let courseDetailClassLi = document.getElementById('courseDetailClass').getElementsByTagName('li')
      if (str === 'catalogue') {
        courseDetailClassLi[1].setAttribute('class', 'activeSelect')
        courseDetailClassLi[0].setAttribute('class', '')
        this.$refs.myswiper.swiper.slideTo(1, 300, false)
      } else {
        courseDetailClassLi[0].setAttribute('class', 'activeSelect')
        courseDetailClassLi[1].setAttribute('class', '')
        this.$refs.myswiper.swiper.slideTo(0, 300, false)
      }
    },
    selectTitle (i) {
      this.selectIndex = i
    },
    onPlayerPlay () {
      if (this.firstPlay) {
        this.firstPlay = false
        axios.get(`http://www.gk0101.com/exam/rest/v1/studySheet/saveWapStudySheet?courseId=${this.$route.query.courseId}&knowPointId=${this.knowPointId}&userId=${this.userID}&institutionId=10103`).then(res => {
          const result = res.data.data
          this.studyData = result
        })
        axios.get(`http://www.gk0101.com/exam/rest/v1/studyProgress/getWapStudyProgress?courseId=${this.$route.query.courseId}&userId=${this.userID}&institutionId=10103`).then(res => {})
      }
    },
    onPlayerEnded () {
      let URL = `http://www.gk0101.com/exam/rest/v1/studySheet/updateWapStudySheet?studySheetId=${this.studyData}&institutionId=10103`
      let URL2 = `http://www.gk0101.com/exam/rest/v1/studyProgress/saveWapStudyProgress?courseId=${this.$route.query.courseId}&knowPointId=${this.knowPointId}&userId=${this.userID}&institutionId=10103`
      axios.get(URL).then(res => {})
      axios.get(URL2).then(res => {})
    },
    selectChapter (i) {
      if (!this.userID) {
        if (i !== this.knowPointId) {
          // 跳转登录页
          return this.$router.push('/mine')
        } else {
          axios.get(`http://www.gk0101.com/resource/rest/v1/file/getVideo?knowPointId=${i}`).then(res => {
            const result = res.data.data[0]
            const videoSrc = result.saveFilePath + '/' + result.saveFileName
            this.$refs.videoPlayer.player.src('http://www.gk0101.com' + videoSrc)
          })
        }
      } else {
        if (!this.isBuy && i !== this.knowPointId) {
          // 没有购买且点击其他知识点
          return Dialog.alert({message: '请先购买课程！', confirmButtonColor: '#3246d8', closeOnPopstate: true, closeOnClickOverlay: true})
        } else {
          this.selectChapterIndex = i
          this.knowPointId = i
          axios.get(`http://www.gk0101.com/resource/rest/v1/file/getVideo?knowPointId=${i}`).then(res => {
            const result = res.data.data[0]
            const videoSrc = result.saveFilePath + '/' + result.saveFileName
            this.$refs.videoPlayer.player.src('http://www.gk0101.com' + videoSrc)
          })
          let URL = `http://www.gk0101.com/exam/rest/v1/studySheet/saveWapStudySheet?courseId=${this.$route.query.courseId}&knowPointId=${i}&userId=${this.userID}&institutionId=10103`
          axios.get(URL).then(res => {
            const result = res.data.data
            this.studyData = result
          })
        }
      }
    }
  },
  created () {
    let courseId = this.$route.query.courseId
    // 获取课程信息列表
    axios.get(`http://www.gk0101.com/user/study/showKnowPointList?courseId=${courseId}&equipmentType=3&institutionId=10103&userId=${this.userID}`).then(res => {
      const result = res.data.data
      this.knowPointsTotalList = result.knowPoints
      this.knowPointsTitleList = this.knowPointsTotalList.filter(item => item.knowPointType === 4)
      this.knowPointsTotalList.forEach((item, index) => {
        if (item.knowPointType === 4) {
          this.knowPointsTitleListIndex.push(index)
        }
      })
      for (let i = 0; i < this.knowPointsTitleList.length; i++) {
        this.knowPointsChapterList[i] = this.knowPointsTotalList.slice(this.knowPointsTitleListIndex[i] + 1, this.knowPointsTitleListIndex[i + 1])
      }
      this.totalMic = this.knowPointsTotalList.length - this.knowPointsTitleListIndex.length
      this.courseName = result.courseName
      this.intro = result.intro
      this.teacherName = result.teacherName
      if (result.courseClassify === 1) {
        this.courseClassify = '公共课程'
      } else if (result.courseClassify === 3) {
        this.courseClassify = '基础课程'
      } else if (result.courseClassify === 7) {
        this.courseClassify = '专业课程'
      }
      if (result.studyCount) {
        this.studyCount = result.studyCount
      } else {
        this.studyCount = 0
      }
      if (result.price !== 0) {
        this.price = result.price
      }
      if (this.userID) {
        // 判断用户是否已经购买
        let params = new URLSearchParams()
        params.append('institutionId', '10103')
        params.append('courseId', courseId)
        params.append('buyerUserId', this.userID)
        axios.post(`http://www.gk0101.com/user/rest/v1/order/checkCourseIsBuy`, params).then(res => {
          const ret = res.data.data
          if (ret.length === 0) {
            // 用户没有购买该课程
            this.isBuy = false
          } else {
            // 用户已经购买该课程
            this.isBuy = true
            this.btnText = '立即学习'
          }
        })
        // 得到学习进度
        let URL = `http://www.gk0101.com/exam/rest/v1/studyProgress/getWapStudyProgress?courseId=${courseId}&userId=${this.userID}&institutionId=10103`
        axios.get(URL).then(res => {
          if (res.data.code === 0) {
            const result = res.data.data
            this.knowPointId = this.selectChapterIndex = result.knowPointId
            this.selectIndex = result.chapterId
            // 通过knowPointId得到相关视频信息
            axios.get(`http://www.gk0101.com/resource/rest/v1/file/getVideo?knowPointId=${this.knowPointId}`).then(res => {
              const result = res.data.data[0]
              const videoSrc = result.saveFilePath + '/' + result.saveFileName
              this.$refs.videoPlayer.player.src('http://www.gk0101.com' + videoSrc)
            })
          }
        })
        // 得到学过的课程
        axios.get(`http://www.gk0101.com/exam/rest/v1/studySheet/getLearnedKnowPointList?userId=${this.userID}&courseId=${courseId}`).then(res => {
          const ret = res.data.data
          for (const item of ret) {
            this.learnedCourse.push(item.knowPointId)
          }
        })
      } else {
        // 获取第一节知识点视频
        this.knowPointId = this.selectChapterIndex = this.knowPointsChapterList[0][0].id
        axios.get(`http://www.gk0101.com/resource/rest/v1/file/getVideo?knowPointId=${this.knowPointId}`).then(res => {
          const result = res.data.data[0]
          const videoSrc = result.saveFilePath + '/' + result.saveFileName
          this.$refs.videoPlayer.player.src('http://www.gk0101.com' + videoSrc)
        })
      }
    })
  },
  mounted () {
    document.getElementsByClassName('vjs-play-control')[0].className = 'vjs-play-control vjs-control vjs-button vjs-playing needsclick'
  }
}
</script>

<style lang="scss">
  .coursedetail{
    .video-player{
      width:750px;
      height:422px;
      .video-js{
        &>.vjs-big-play-button{
          width:144px;
          height:144px !important;
          transform: translate(-50%,-50%);
          margin-left: 0 !important;
          margin-top: 0 !important;
          background-color: none !important;
          background: url('../assets/imgs/live_play@2x.png') center center no-repeat;
          background-size: 100% 100%;
          border: none;
          outline: none;
          span{
            display: none;
          }
        }
      }
      .vjs-control-bar{
        height: 88px;
        .vjs-play-control{
          width:53px;
          height:53px;
          background: url('../assets/imgs/live_play@2x.png') center center no-repeat;
          border: none;
          outline: none;
          background-size: 100% 100%;
          margin: 13px 0 11px 15px;
          span{
            display: none;
          }
        }
        .vjs-volume-panel{
          display: none;
        }
      }
      .vjs-current-time{
        width:54px;
        height:28px;
        font-size:20px;
        font-family:PingFangSC-Regular;
        font-weight:400;
        color:rgba(255,255,255,1);
        line-height:28px;
        margin: 0 0 !important;
        position: absolute;
        bottom: 10px;
        left: 88px;
        text-align: center;
      }
      .vjs-duration{
        position: absolute;
        margin: 0 !important;
        width:50px;
        height:28px;
        font-size:20px;
        font-family:PingFangSC-Regular;
        font-weight:400;
        color:rgba(255,255,255,1);
        line-height:28px;
        text-align: center;
        bottom: 10px;
        right: 96px;
      }
      .vjs-progress-control{
          width: 560px !important;
          flex: none;
          margin-left: 26px;
        .vjs-progress-holder{
          background:rgba(226,226,226,1);
          margin-top: -6px;
          .vjs-play-progress::before{
            width:24px;
            height:24px;
            background:rgba(255,255,255,1);
            border-radius:12px;
            top: -10px;
          }
        }
      }
      .vjs-fullscreen-control{
        width:36px;
        height:36px;
        border: none;
        outline: none;
        background: url('../assets/imgs/play_full@2x.png') center center no-repeat;
        background-size: 100% 100%;
        position: absolute;
        right: 30px;
        top: 26px;
        span{
          display: none;
        }
      }
    }
    .swiper-container{
      background-color: #fff;
      height: 540px;
      overflow-y: scroll;
      .swiper-slide{
        padding-top: 24px;
        padding-left: 24px;
        .courseDetail_courseName{
          display: block;
          height:48px;
          font-size:34px;
          font-family:PingFangSC-Regular;
          font-weight:400;
          color:rgba(0,0,0,1);
          line-height:48px;
        }
        .courseDetail_courseintro{
          margin-top: 22px;
          width:704px;
          line-height:40px;
          font-size:28px;
          font-family:PingFangSC-Regular;
          font-weight:400;
          color:rgba(153,153,153,1);
          span{
            font-size:28px;
            font-family:PingFangSC-Regular;
            font-weight:400;
            color:rgba(153,153,153,1);
          }
        }
        .courseDetail_coursetotalInfo{
          margin-top: 24px;
          width:704px;
          height:56px;
          box-sizing: border-box;
          margin-bottom: 54px;
          line-height: 56px;
          padding: 0 32px;
          span{
            font-size:24px;
            font-family:PingFangSC-Regular;
            font-weight:400;
            color:rgba(102,102,102,1);
            margin-right: 58px;
            &.courseDetail_coursetotalMic{
              margin-right: 0;
            }
          }
        }
        .courseknowPoints{
          li{
            width: 684px;
            margin-bottom: 48px;
            position: relative;
            .point{
              display: inline-block;
              width:24px;
              height:24px;
              border-radius:36px;
              background:rgba(242,242,242,1);
              position: absolute;
              left: 0;
              top: 12px;
            }
            .selectPoint{
              background: #666666;
            }
            .knowPointName{
              display: inline-block;
              width: 560px;
              font-size:34px;
              font-family:PingFangSC-Regular;
              font-weight:400;
              color:rgba(102,102,102,1);
              line-height:48px;
              margin-left: 58px;
            }
            .select{
              color: #000000;
            }
            img{
              width: 48px;
              height: 48px;
              position: absolute;
              right: 0;
              top: 0;
            }
            .courseknowPointsVideos{
              margin-left: 58px;
              margin-top: 32px;
              li{
                width: 634px;
                height: 48px;
                margin-bottom: 46px;
                line-height: 48px;
                position: relative;
                .icon_video{
                  position: absolute;
                  left: 0;
                  top: 50%;
                  transform: translateY(-50%);
                  width:39px;
                  height:39px;
                }
                span{
                  margin-left: 47px;
                  height:34px;
                  font-size:24px;
                  width: 420px;
                  box-sizing: border-box;
                  display: inline-block;
                  font-family:PingFangSC-Regular;
                  font-weight:400;
                  color: black;
                  line-height:34px;
                  &.learnedKnowPoint{
                    color:rgba(102,102,102,1);
                  }
                  &.selectChapter{
                    color: #3246D8;
                  }
                }
                button{
                  position: absolute;
                  right: 0;
                  top: 0;
                  font-size:24px;
                  font-family:PingFangSC-Regular;
                  font-weight:400;
                  color:rgba(50,70,216,1);
                  width:154px;
                  height:48px;
                  border-radius:36px;
                  border:2px solid rgba(50,70,216,1);
                  outline: none;
                  background: #fff;
                }
              }
            }
          }
        }
      }
    }
  }
  .van-dialog{
    width: 60%;
    height: 200px;
    .van-dialog__message{
      font-size: 28px;
      height: 70px;
    }
    .van-button{
      font-size: 28px;
      border-top: 1px solid #999; /*no*/
      padding-top: 15px;
    }
  }
</style>
<style lang="scss" scoped>
  .coursedetail{
    position: relative;
    background:rgba(242,242,242,1);
    .backArrow{
      position: absolute;
      left: 38px;
      top: 66px;
      z-index: 2;
      img{
        width: 48px;
        height: 48px;
      }
    }
    .signOutState{
      width:750px;
      height:422px;
      background-color: rgb(24,58,83);
      text-align: center;
      line-height: 422px;
      span{
        color: #fff;
        font-size:34px;
        font-family:PingFangSC-Regular;
        font-weight:400;
      }
    }
    .courseDetailClass{
      height: 98px;
      display: flex;
      background:rgba(255,255,255,1);
      align-items: center;
      justify-content: space-around;
      li{
        height: 98px;
        line-height: 98px;
        box-sizing: border-box;
        font-size:34px;
        font-family:PingFangSC-Regular;
        font-weight:400;
        color:rgba(153,153,153,1);
        padding-bottom: 24px;
        &.activeSelect{
          border-bottom: 4px solid rgba(102,102,102,1);
        }
      }
    }
    .courseDetailfooter{
      margin-top: 20px;
      width: 750px;
      box-sizing: border-box;
      height: 208px;
      background-color: #fff;
      padding-left: 24px;
      span{
        display: block;
        &:nth-child(1){
          font-size:34px;
          margin-top: 24px;
          font-family:PingFangSC-Regular;
          font-weight:400;
          color: #3246D8;
        }
        &:nth-child(2){
          margin-top: 10px;
          font-size:28px;
          font-family:PingFangSC-Regular;
          font-weight:400;
          color:rgba(153,153,153,1);
        }
      }
      button{
        position: fixed;
        bottom: 0;
        left: 0;
        width:750px;
        height:104px;
        background:#3246d8;
        line-height: 104px;
        text-align: center;
        border: none;
        outline: none;
        font-size:34px;
        font-family:PingFangSC-Regular;
        font-weight:400;
        color:rgba(255,255,255,1);
      }
    }
  }

</style>
