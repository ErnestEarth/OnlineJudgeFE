<template>
  <div>
    <Panel>
      <div slot="title">{{$t('m.Problems_List')}}</div>
      <div v-if="(allowence || isAdminRole) && contestStatus == 0">
        <Table v-if="contestRuleType == 'ACM' || OIContestRealTimePermission"
              :columns="ACMTableColumns"
              :data="problems"
              @on-row-click="goContestProblem"
              :no-data-text="$t('m.No_Problems')"></Table>
        <Table v-else
              :data="problems"
              :columns="OITableColumns"
              @on-row-click="goContestProblem"
              no-data-text="$t('m.No_Problems')"></Table>
        </div>
    </Panel>
  </div>
</template>

<script>
  import {mapState, mapGetters} from 'vuex'
  import {ProblemMixin} from '@oj/components/mixins'
  import api from '@oj/api'

  export default {
    name: 'ContestProblemList',
    mixins: [ProblemMixin],
    data () {
      return {
        allowence: 1,
        ACMTableColumns: [
          {
            title: '#',
            key: '_id',
            sortType: 'asc',
            width: 150
          },
          {
            title: this.$i18n.t('m.Title'),
            key: 'title'
          },
          {
            title: this.$i18n.t('m.Total'),
            key: 'submission_number'
          },
          {
            title: this.$i18n.t('m.AC_Rate'),
            render: (h, params) => {
              return h('span', this.getACRate(params.row.accepted_number, params.row.submission_number))
            }
          }
        ],
        OITableColumns: [
          {
            title: '#',
            key: '_id',
            width: 150
          },
          {
            title: this.$i18n.t('m.Title'),
            key: 'title'
          }
        ]
      }
    },
    mounted () {
      let data = {
        contest_id: this.$route.params.contestID
      }
      api.getUserEndTime(data).then(res => {
        if (res.data.data.joined === 0 && this.contestStatus !== '-1') {
          this.allowence = 0
        }
      })
      this.getContestProblems()
    },
    methods: {
      getContestProblems () {
        this.$store.dispatch('getContestProblems').then(res => {
          if (this.isAuthenticated) {
            if (this.contestRuleType === 'ACM') {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            } else if (this.OIContestRealTimePermission) {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            }
          }
        })
      },
      goContestProblem (row) {
        this.$router.push({
          name: 'contest-problem-details',
          params: {
            contestID: this.$route.params.contestID,
            problemID: row._id
          }
        })
      }
    },
    computed: {
      ...mapState({
        problems: state => state.contest.contestProblems
      }),
      ...mapGetters(
        ['isAuthenticated', 'contestRuleType', 'OIContestRealTimePermission',
          'contestStatus', 'isAdminRole'])
    }
  }
</script>

<style scoped lang="less">
</style>
