<template>
  <div>
    <el-row class="tle">
      <el-col :span="7">
          <div class="div-quarter">
            <span class="title-main">{{ coInfo.name }}</span>
          </div>
      </el-col>
      <el-col :span="2">
          <div class="blackline"></div>
      </el-col>
      <el-col :span="15">
          <div>
            <span class="title-sup">{{ coInfo.semester }}</span>
          </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col>
        <el-table
        :data="coState"
        v-loading="loading"
        style="width: 100%"
        stripe>
        <el-table-column label="NAME" align="center">
          <template slot-scope="scope" >
            <el-button @click="getpath(scope)" class="name">{{ scope.row.name }}</el-button>
          </template>
        </el-table-column>
        <el-table-column
          label="STATUS"
          align="center"
          width="150">
          <template slot-scope="scope">
            <el-button :style="colors(scope.row)" @click="updateAss(scope.row)" class="fake-href">
              {{ getScore(scope.row) }}
            </el-button>
          </template>
        </el-table-column>
        <el-table-column
          prop="release_date"
          align="center"
          label="RELEASE">
        </el-table-column>
        <el-table-column
          prop="deadline"
          align="center"
          label="DUE">
        </el-table-column>
          <el-table-column
          label="SCOREBOARD"
          align="center">
            <template slot-scope="scope">
              <el-button @click="goToSB(scope.row)" size="small">check</el-button>
            </template>
          </el-table-column>
          <el-table-column
          label="SUBMISSION"
          align="center">
            <template slot-scope="scope">
              <el-button @click="goToSU(scope.row)" size="small">check</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
    <el-row class="row-half" v-if="show">
      <el-col>
        <el-tooltip class="item" effect="dark" content="A list for those who made some mistakes" placement="left">
          <el-button size="mini" @click="showPending" class="button-shot">Show list</el-button>
        </el-tooltip>
      </el-col>
    </el-row>
    <el-row v-if="show">
      <el-col>
        <el-table
        :data="pendingList"
        class="table-only"
        stripe>
        <el-table-column
          prop="commit_tag"
          label="commit_tag">
        </el-table-column>
        <el-table-column
          prop="course_id"
          label="course id">
        </el-table-column>
        <el-table-column
          prop="submission_time"
          label="submission time">
        </el-table-column>
        <el-table-column
          prop="submitter"
          label="submitter">
        </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import { mapState, mapGetters } from 'vuex'

export default {
  data () {
    return {
      coState: [],
      coInfo: {
      },
      show: false,
      pendingList: [{
        commit_tag: '',
        course_id: '',
        submission_time: 0,
        submitter: ''
      }],
      length: 0,
      loading: true
    }
  },
  methods: {
    getpath (scope) {
      window.location.href = scope.row.descr_link
    },
    colors (situation) { // don't use state as the variable name
      const grade = situation.score / situation.overall_score
      let bgColor = 'black'
      if (grade >= 0.7) {
        bgColor = '#67C23A'
      } else if (grade >= 0.5) {
        bgColor = '#E6A23C'
      } else if (grade >= 0.2) {
        bgColor = '#F56C6C'
      } else if (grade === null || grade === undefined) {
        bgColor = '#909399'
      }
      return `background-color:${bgColor}; color:white; width:100px;opacity: 0.7;`
    },
    showPending () {
      this.show = !this.show
    },
    updateAss (info) {
      this.$store.commit('updateAss', info)
      this.$router.push(`${this.$route.path}/submission/${info.name}`)
    },
    getScore (row) {
      if (row.score === null || row.score === undefined) {
        return 'no result'
      } else {
        return `${row.score}/${row.overall_score}`
      }
    },
    goToSB (row) {
      this.$router.push(`/course/${this.$route.params.course_code}/scoreboard/${row.name}`)
    },
    goToSU (row) {
      this.updateAss(row)
    },
    getCoState (data) {
      let that = this
      if (!data) {
        return data
      }
      let result = []
      data.map(function (a) {
        if (a.deadline) {
          a.deadline = that.formatUTC(a.deadline)
        } else {
          a.deadline = '/' // no time!
        }
        if (a.release_date) {
          a.release_date = that.formatUTC(a.release_date)
        } else {
          a.release_date = '/' // no time!
        }
        result.push(a)
      })
      return result
    },
    formatUTC (utcdatetime) {
      let Tpos = utcdatetime.indexOf('T')
      let Zpos = utcdatetime.indexOf('Z')
      let yearmonthday = utcdatetime.substr(0, Tpos)
      let hourminutesecond = utcdatetime.substr(Tpos + 1, Zpos - Tpos - 1)
      let newdatetime = yearmonthday + ' ' + hourminutesecond
      let timestamp = new Date(Date.parse(newdatetime))
      timestamp = timestamp.getTime()
      timestamp = timestamp / 1000
      timestamp = timestamp + 8 * 60 * 60
      let date = new Date(parseInt(timestamp) * 1000)
      let hours = date.toString().split(' ')[4]
      let days = date.toLocaleString().split(' ')[0]
      return `${days} ${hours}`
    },
    filterUid (code) {
      let uid = this.codeToUid({
        code: code,
        cate: 'course'
      })
      if (uid === '') {
        this.$router.push({
          name: 'notFound'
        })
      } else {
        return uid
      }
    }
  },
  mounted () {
    this.coInfo = this.getCoInfo
  },
  created () {
    if (this.getAuth) {
      this.axios.get(`${this.Api}/student/${this.getID}/course/${this.filterUid(this.$route.params.course_code)}/assignment/`)
        .then((response) => {
          this.length = response.data.length
          this.coState = this.getCoState(response.data)
          this.$store.commit('loadAss', this.coState)
          this.loading = false
        }).catch((err) => {
          this.$message({
            type: 'error',
            message: err,
            showClose: true
          })
        })
    }
    if (!this.getAuth) {
      this.axios.get(`${this.Api}/course/${this.getUid}/queue`)
        .then((response) => {
          this.pendingList = response.data
        }).catch((err) => {
          this.$message({
            type: 'error',
            message: err,
            showClose: true
          })
        })
    }
  },
  computed: {
    ...mapGetters([
      'codeToUid'
    ]),
    ...mapState({
      getAuth: state => state.isAuthorized,
      getID: state => state.baseInfo.uid,
      getUid: state => state.coInfo.uid,
      getCoInfo: state => state.coInfo,
      Api: state => state.api
    })
  }
}
</script>
<style scoped>
  .blackline {
    height: 30px;
    background-color: black;
    width: 1px;
    margin-top: 5px;
    margin-right: 5%;
  }
  .tle {
    display: flex;
    flex-direction: row;
    margin: 4% 0 10px 0;
  }
  .name{
    border: none!important;
    padding: 0 0 2px 0!important;
  }
  .div-quarter {
    margin-right: 5%;
  }
  .title-main {
    font-size: 30px;
  }
  .title-sup {
    font-size: 35px;
  }
  .fake-href {
    text-decoration: none;
    color: #cb0c0a;
  }
  .button-shot {
    float: right;
  }
  .table-only {
    width: 100%;
  }
  .row-half{
    margin-top: 10%;
  }
</style>
