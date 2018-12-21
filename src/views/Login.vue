<template>
  <v-app>
    <v-container fluid id="background"></v-container>
    <v-container fluid fill-height class="login">
      <v-layout align-center justify-center>
        <v-flex xs12 sm8 md4>
          <!-- Display login for person who don't login yet. -->
          <v-card v-if="alreadyLogin == false" class="login-block">
            <v-card-text>
              <img class="logo-img" src="https://www.sit.kmutt.ac.th/en/wp-content/uploads/2018/05/logo-flat-blk.png"/>
              <v-form>
                <v-text-field v-model="username" prepend-icon="person" name="login" label="Login" type="text" color="black" ></v-text-field>
                <v-text-field v-model="password" prepend-icon="lock" name="password" label="Password" id="password" color="black" type="password" @keyup.enter.native="loginAuthen()"></v-text-field>
              </v-form>
            </v-card-text>
              <v-btn
               :loading="loading"
               :disabled="loading"
               color="green lighten-1" @click="loginAuthen()">
                Login
                <span slot="loader" class="custom-loader">
                  <v-icon light>cached</v-icon>
                </span>
              </v-btn>
              <hr/>
          </v-card>
          <!-- Detail for person who already login-->
          <v-card v-else class="login-block">
            <v-card-text>
              <img class="logo-img" src="https://www.sit.kmutt.ac.th/en/wp-content/uploads/2018/05/logo-flat-blk.png"/>
              <h2>Profile Detail : {{getUser.userName}} {{getUser.userId}}</h2>
            </v-card-text>
              <router-link to="/" >
                <v-btn color="green lighten-1" @click="setIsShowToolBar(true)">Back To Homepage</v-btn>
              </router-link>
                <v-btn color="green lighten-1" @click="verifyJWT()">Send JWT To Verify</v-btn>
              <v-btn color="green lighten-1)" @click="logout()">Logout</v-btn>
              <hr/>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import axios from 'axios'
import jwtDecode from 'jwt-decode'
import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'login',
  data () {
    return {
      profile: {},
      alreadyLogin: false,
      authorized: false,
      username: '',
      password: '',
      loading: false
    }
  },
  computed: {
    ...mapGetters(['getUser', 'getJwtToken', 'getUser'])
  },
  mounted () {
    this.setIsShowToolBar(false)
    this.verifyIsLoginByJwtYet()
  },
  methods: {
    ...mapActions(['setJwtToken', 'setUser','setIsShowToolBar']),
    verifyIsLoginByJwtYet: function () {
      let jwtToken = localStorage.getItem('jwtToken')
      if (jwtToken != null) {
        this.alreadyLogin = true
      }
    },
    loginAuthen: async function () {
      this.loading = true
      let id = this.username
      let password = this.password
      let userAuthentication = await axios.post(process.env.VUE_APP_USER_SERVICE_URL + '/user/login',
        {
          id,
          password
        }
      ).catch((error)=>{
          this.$swal('กรุณา login ใหม่', error.response.data.message , 'error');
          this.loading = false
      })
      userAuthentication = userAuthentication.data
      let jwtTokenLocalStorage = localStorage.getItem('jwtToken')
      if (userAuthentication != null) {
        localStorage.setItem('jwtToken', userAuthentication.jwtToken)
        let userAfterDecodeJWT = jwtDecode(userAuthentication.jwtToken).user
        this.setJwtToken(userAuthentication.jwtToken)
        this.setUser(userAfterDecodeJWT)
        this.setIsShowToolBar(true)
        this.$emit('loadToolBarAfterLogin')
        this.$router.push('/')
      }
    },
    logout: function (){
      localStorage.removeItem('jwtToken')
      this.alreadyLogin = false
    },
    verifyJWT: async function(){
      let jwtTokenLocalStorage = localStorage.getItem('jwtToken')
      let verifyData = await axios.get(
        `${process.env.VUE_APP_USER_SERVICE_URL}/users`,
        {
          headers: {
            Authorization: `Bearer ${jwtTokenLocalStorage}`
          }
        }
      )
      verifyData = verifyData
    }
  }
}
</script>

<style scoped>
  #background {
    background-image: url("https://scontent.fbkk7-2.fna.fbcdn.net/v/t1.0-9/64916_10150597397233789_1395936545_n.jpg?_nc_cat=109&_nc_ht=scontent.fbkk7-2.fna&oh=466d1b476103f596fab9fd67344f6155&oe=5C961FFA");
    background-size: 100%;
    filter: blur(5px);
  }

  .login {
    position: absolute;
  }

  .login-block {
    border-radius: 10px;
  }

  .logo-img {
    width: 120px;
    height: auto;
  }

  .v-btn {
    width: 80%;
  }

  .custom-loader {
    animation: loader 1s infinite;
    display: flex;
  }
  @-moz-keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @-webkit-keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @-o-keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
</style>

