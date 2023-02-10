<template>
  <Row type="flex" flex-direction="column" flex-wrap="wrap" justify="space-around">
    <Col :span="20" id="status">
      <Alert :type="status.type">
        <span class="title">{{$t('m.' + status.statusName.replace(/ /g, "_"))}}</span>
        <!-- <div>{{ submission}}</div> -->
        <div slot="desc" class="content">
          <template v-if="isCE">
            <pre>{{submission.statistic_info.err_info}}</pre>
          </template>
          <template v-else>
            <span>{{$t('m.Time')}}: {{submission.statistic_info.time_cost | submissionTime}}</span>
            <span>{{$t('m.Memory')}}: {{submission.statistic_info.memory_cost | submissionMemory}}</span>
            <span>{{$t('m.Lang')}}: {{submission.language}}</span>
            <span>{{$t('m.Author')}}: {{submission.username}}</span>
          </template>
        </div>
      </Alert>
    </Col>

    <!--后台返info就显示出来， 权限控制放后台 -->
    <Col v-if="submission.info && !isCE && !isRE && !isSE && !isPending" :span="20" id="test_case">
      <div id="title">测试点信息</div>
      <div v-if="status.statusName!='Accepted'" style="color: black; font-size: 16px; margin-left: 8px; margin-bottom: 8px;">
        提供第一个非 AC 测试点的下载
        <Button type="warning" :loading="submitting" @click="downloadTestcase()" style="margin-left: 8px;">
          <span>下载</span>
        </Button>
      </div>
      <div v-for="item in submission.info.data">
        <div id="test_case_info" v-if="item.result==0" style="background: green;">
          <div style="font-size: 18px; margin-left: 5px; align-self: flex-start;">{{item.test_case}}</div>
          <div id="content" v-if="item.result==0">AC</div>
          <div>{{item.cpu_time}} ms / {{(parseInt(item.memory) / 1048576).toFixed(1)}} MB</div>
        </div>
        <div id="test_case_info" v-else style="background: red;">
          <div style="font-size: 18px; margin-left: 5px; align-self: flex-start;">{{item.test_case}}</div>
          <div id="content" v-if="item.result==-1">WA</div>
          <div id="content" v-else-if="item.result==1">TLE</div>
          <div id="content" v-else-if="item.result==2">TLE</div>
          <div id="content" v-else-if="item.result==3">MLE</div>
          <div>{{item.cpu_time}} ms / {{(parseInt(item.memory) / 1048576).toFixed(1)}} MB</div>
        </div>
      </div>
    </Col>

    <!-- <Col v-if="submission.info && !isCE" :span="20">
      <Table stripe :loading="loading" :disabled-hover="true" :columns="columns" :data="submission.info.data"></Table>
    </Col> -->

    <Col :span="20">
      <Highlight :code="submission.code" :language="submission.language" :border-color="status.color"></Highlight>
    </Col>
    <!-- <Col v-if="submission.can_unshare" :span="20">
      <div id="share-btn">
        <Button v-if="submission.shared"
                type="warning" size="large" @click="shareSubmission(false)">
          {{$t('m.UnShare')}}
        </Button>
        <Button v-else
                type="primary" size="large" @click="shareSubmission(true)">
          {{$t('m.Share')}}
        </Button>
      </div>
    </Col> -->
  </Row>

</template>

<script>
  import api from '@oj/api'
  import utils from '@/utils/utils'
  import {JUDGE_STATUS} from '@/utils/constants'
  // import utils from '@/utils/utils'
  import Highlight from '@/pages/oj/components/Highlight'

  export default {
    name: 'submissionDetails',
    components: {
      Highlight
    },

    data () {
      return {
        submission: {
          result: '0',
          code: '',
          info: {
            data: []
          },
          statistic_info: {
            time_cost: '',
            memory_cost: ''
          }
        },
        isConcat: false,
        loading: false
      }
    },
    mounted () {
      this.getSubmission()
    },
    methods: {
      getSubmission () {
        this.loading = true
        api.getSubmission(this.$route.params.id).then(res => {
          this.loading = false
          this.submission = res.data.data
        }, () => {
          this.loading = false
        })
      },
      downloadTestcase () {
        for (let item of this.submission.info.data) {
          if (item.result !== 0) {
            let url = '/admin/test_case?problem_id=' + this.submission.problem + '&test_case=' + item.test_case
            utils.downloadFile(url)
            break
          }
        }
      }
    },
    computed: {
      status () {
        return {
          type: JUDGE_STATUS[this.submission.result].type,
          statusName: JUDGE_STATUS[this.submission.result].name,
          color: JUDGE_STATUS[this.submission.result].color
        }
      },
      isCE () {
        return this.submission.result === -2
      },
      isRE () {
        return this.submission.result === 4
      },
      isSE () {
        return this.submission.result === 5
      },
      isPending () {
        return this.submission.result === 6
      },
      isAdminRole () {
        return this.$store.getters.isAdminRole
      }
    }
  }
</script>

<style scoped lang="less">
  #status {
    .title {
      font-size: 20px;
    }
    .content {
      margin-top: 10px;
      font-size: 14px;
      span {
        margin-right: 10px;
      }
      pre {
        white-space: pre-wrap;
        word-wrap: break-word;
        word-break: break-all;
      }
    }
  }
  #test_case {
    padding: 10px;
    border-radius: 5px;
    // border: 1px solid;
    background-color: rgb(248, 248, 248);
    #title {
      font-size: 20px;
      color: black;
      margin-bottom: 10px;
      margin-left: 8px;
    }
    #test_case_info {
      display: flex;
      flex-direction: column;
      align-items: center;
      float: left;
      width: 100px;
      height: 100px;
      margin-left: 8px;
      margin-bottom: 8px;
      color: white;
      overflow: visible !important;
      #content {
        font-size: 32px;
      }
    }
  }

  .admin-info {
    margin: 5px 0;
    &-content {
      font-size: 16px;
      padding: 10px;
    }
  }

  #share-btn {
    float: right;
    margin-top: 5px;
    margin-right: 10px;
  }

  pre {
    border: none;
    background: none;
  }
</style>
