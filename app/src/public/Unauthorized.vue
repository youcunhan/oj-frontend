<template>
  <div class="wall-main">
    <el-container style="height: 100%">
      <el-main style="height: 100%">
        <el-row type="flex" justify="center" align="middle">
          <el-col style="width: auto">
            <span class="title-number">4 0 1</span>
          </el-col>
        </el-row>
        <el-row type="flex" justify="center" align="middle">
          <el-col :span="10">
            <el-card class="cards">
              <el-row type="flex" justify="space-around" align="middle">
                <el-col class="space-center">
                  <span class="words">Unauthorized request or login was out of date, please login again!</span>
                </el-col>
              </el-row>
              <el-row type="flex" justify="space-around" align="middle">
                <img v-bind:src="img2" class="img">
              </el-row>
              <el-row type="flex" justify="space-around" align="middle">
                <el-button @click="login"><span class="title-button">Login</span></el-button>
              </el-row>
            </el-card>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>
<script>
import na from './Navigation'
import { mapState } from 'vuex'

export default {
  data () {
    return {
      img2: require('../assets/pagenotfound.jpg')
    }
  },
  components: {
    'v-na': na
  },
  computed: mapState({
    getAuth: state => state.isAuthorized,
    Api: state => state.api
  }),
  methods: {
    login () {
      this.axios({
        method: 'get',
        url: `${this.Api}/user/login/oauth/param`
      }).then((response) => {
        this.$store.commit('login')
        window.location.href = response.data.login_url
      }).catch((err) => {
        this.$message({
          type: 'error',
          message: err,
          showClose: true
        })
      })
    }
  }
}
</script>
<style scoped>
  .wall-main {
    height: 100vh;
  }

  .words {
    color: black;
    font-size: 40px;
    flex-wrap: nowrap !important;
  }
  .cards {
    padding: 4rem 0;
  }

  .img {
    padding: 3rem 0;
    height: 200px !important;
  }
  .space-center {
    margin: 0 5% 0 10%;
  }
  .title-button {
    font-size: 25px;
  }
  .title-number {
    color: #b82e3b;
    font-size: 7rem;
  }
</style>
