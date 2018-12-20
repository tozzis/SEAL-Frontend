<template>
  <v-app>
      <div class="container-fluid over-scroll">
        <div class="row">
          <div class="col-8">
            <div class="row">
              <div class="col-12">
                <d-player class="video-player" @play="play" :options="config" ref="player"></d-player>
              </div>
              <div class="col-12">
                <p class="video-name">{{videoDetail.video_name}}</p>
                <p class="descript-video">
                  <v-icon class="icon-description">person</v-icon>
                  <!-- {{videoDetail.teacher.teacher_name}} fdsfsd -->
                  {{videoDetail.teacherName}}
                </p>
                <p class="descript-video">
                  <v-icon class="icon-description">access_time</v-icon>
                  {{videoDetail.videoDate}}
                </p>
                <p class="descript-video">
                  <v-icon class="icon-description">location_on</v-icon>
                  {{videoDetail.roomName}}
                </p>

            <div>
              <div class="row">
                  <div class="col-1">
                      <img class="circle" :src="`${getUser.userImg}`"/>
                  </div>
                  <div class="col-11">
                  <v-text-field
                      label="Comment"
                      placeholder="comment.."
                      box dark color="white"
                      v-model="post_comment"
                      @keyup.enter.native="PostComment()"
                  ></v-text-field>
                  </div>
                  <div class="col-12" v-if="comment.length>0">
                      <v-list two-line dark>
                          <template v-for="item in comment">
                              <v-list-tile :key="item.id" avatar>
                              <v-list-tile-avatar>
                                  <img :src="`${item.user.image}`"/>
                              </v-list-tile-avatar>
                              <v-list-tile-content>
                                  <v-list-tile-title>{{item.user.firstname}} {{item.user.lastname}}</v-list-tile-title>
                                  <v-list-tile-sub-title>{{item.comment}}</v-list-tile-sub-title>
                              </v-list-tile-content>
                              <v-list-tile-action>
                                <v-icon color="grey lighten-1"></v-icon>
                                <v-list-tile-action-text>{{item.updatedAt}}</v-list-tile-action-text>
                              </v-list-tile-action>
                              </v-list-tile>
                          </template>
                      </v-list>
                  </div>
              </div>
          </div>

              </div>
            </div>
          </div>
          <div class="col-4">
            <div class="row">
              <div class="col-12">
                <video-suggest class="suggestion"/>
              </div>
            </div>
          </div>
        </div>
      </div>
  </v-app>
</template>
<script>
import axios from 'axios'
import VueDPlayer from 'vue-dplayer'
import { mapActions, mapGetters } from 'vuex'
import 'vue-dplayer/dist/vue-dplayer.css'
import DisplayVideoListSuggest from '../components/Content/DisplayVideoCardSuggest.vue'

export default {
  name: 'PlayVideo',
  mounted () {
    if (localStorage.getItem('jwtToken') == null) {
      this.$router.push({ path: '/login' })
    } else {
      this.fetchVideoDetail(),
      this.getComment()
    }
  },
  components: {
    'd-player': VueDPlayer,
    'video-suggest': DisplayVideoListSuggest
  },
  props: {},
  data () {
    return {
      videoID: 0,
      videoDetail: {
        teacherName: '',
        videoDate: '',
        roomName: '',
        videoUrl: ''
      },
      config: {
        screenshot: true,
        video: {
          pic: 'demo.jpg',
          thumbnails: 'thumbnails.jpg'
        },
        subtitle: {

        },
        logo: require('../assets/seal-logo.png'),
        theme: 'red'
      },
      comment:[],
      post_comment:null
    }
  },
  computed: {
    ...mapGetters(['getUser'])
  },
  methods: {
    getComment: async function () {
        let jwtTokenLocalStorage = localStorage.getItem('jwtToken')
        this.videoID = this.$route.params.videoID
        const comment = await axios.get(
            `${process.env.VUE_APP_VIDEO_SERVICE_URL}/comments/video/${this.videoID}`,
            {
                headers: {
                    Authorization: `Bearer ${jwtTokenLocalStorage}`
                }
            }
        ).catch((response)=>{
            localStorage.removeItem('jwtToken')
            this.$swal('กรุณา login', 'หมดเวลาการใช้งาน', 'error');
            this.$router.push('/login')
        })
        this.comment = comment.data
    },
    PostComment: async function () {
        let jwtTokenLocalStorage = localStorage.getItem('jwtToken')
        this.videoID = this.$route.params.videoID
        await axios.post(
            `${process.env.VUE_APP_VIDEO_SERVICE_URL}/comments/video/${this.videoID}`,{
              userId: this.getUser.userId,
              comment: this.post_comment
            }
        ).then(()=>{
          this.getComment()
        })
        .catch((response)=>{
            localStorage.removeItem('jwtToken')
            this.$swal('กรุณา login', 'หมดเวลาการใช้งาน', 'error');
            this.$router.push('/login')
        })
    },
    fetchVideoDetail: async function () {
      let jwtTokenLocalStorage = localStorage.getItem('jwtToken')
      this.videoID = this.$route.params.videoID
      let videoDetail = await axios.get(
        `${process.env.VUE_APP_VIDEO_SERVICE_URL}/video/${this.videoID}`,
        {
          headers: {
            Authorization: `Bearer ${jwtTokenLocalStorage}`
          }
        }
      ).catch((response)=>{
        localStorage.removeItem('jwtToken')
        this.$swal('กรุณา login', 'หมดเวลาการใช้งาน', 'error');
        this.$router.push('/login')
      })
      videoDetail = videoDetail.data
      this.videoDetail.teacherName = videoDetail.teacher.teacher_name
      this.videoDetail.videoDate = videoDetail.video_date
      this.videoDetail.roomName = videoDetail.room.room_name
      this.videoDetail.videoUrl = videoDetail.player.hls_url
      this.$refs.player.dp.switchVideo({
        url: this.videoDetail.videoUrl
      })
    },
    play () {}
  },
  watch: {
    '$route.params.videoID': function (videoID) {
      this.fetchVideoDetail(videoID)
      this.getComment(videoID)
      this.post_comment = null
    }
  }
}
</script>

<style scoped>
.over-scroll {
  overflow-y: scroll;
}

.video-player {
  width: 100%;
  height: 550px;
  margin-top: 80px;
  margin-bottom: 20px;
}

.video-name {
  text-align: left;
  font-size: 25px;
  color: #ffffff;
  font-weight: 500;
}

.icon-description {
  color: #ffffff !important;
  margin-right: 10px;
}

.descript-video {
  font-size: 16px;
  text-align: left;
  color: #ffffff;
}

.comment-section {
  margin-bottom: 50px;
}

.count-comment {
  font-size: 18px;
  font-weight: 500;
  text-decoration: underline;
  color: #ffffff;
  text-align: left;
}

.comment-img {
  border-radius: 50px;
  width: 50px;
  height: 50px;
  overflow: hidden;
  color: #ffffff;
}

.comment-field {
  background-color: #191919;
  color: #ffffff;
  font-size: 16px;
}

.suggestion {
  width: 100%;
  padding: 0px !important;
  margin-top: 60px;
  margin-bottom: 20px;
}

.circle {
  height: 60px;
  width: 63px;
  border-radius: 50%;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}
.comment {
  font-size: 18px;
  font-weight: 500;
}

</style>
