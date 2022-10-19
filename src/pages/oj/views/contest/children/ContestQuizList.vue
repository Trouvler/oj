<template>
  <div>
    <Panel>
      <div slot="title">{{$t('m.Quizs_List')}}</div>
      <Table v-if="contestRuleType == 'ACM' || OIContestRealTimePermission"
             :columns="ACMTableColumns"
             :data="quizs"
             @on-row-click="goContestQuiz"
             :no-data-text="$t('m.No_Quizs')"></Table>
      <Table v-else
             :data="quizs"
             :columns="OITableColumns"
             @on-row-click="goContestQuiz"
             no-data-text="$t('m.No_Quizs')"></Table>
    </Panel>
  </div>
</template>

<script>
  import {mapState, mapGetters} from 'vuex'
  import {QuizMixin} from '@oj/components/mixins'

  export default {
    name: 'ContestQuizList',
    mixins: [QuizMixin],
    data () {
      return {
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
      this.getContestQuizs()
    },
    methods: {
      getContestQuizs () {
        this.$store.dispatch('getContestQuizs').then(res => {
          if (this.isAuthenticated) {
            if (this.contestRuleType === 'ACM') {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            } else if (this.OIContestRealTimePermission) {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            }
          }
        })
      },
      goContestQuiz (row) {
        this.$router.push({
          name: 'contest-quiz-details',
          params: {
            contestID: this.$route.params.contestID,
            quizID: row._id
          }
        })
      }
    },
    computed: {
      ...mapState({
        quizs: state => state.contest.contestQuizs
      }),
      ...mapGetters(['isAuthenticated', 'contestRuleType', 'OIContestRealTimePermission'])
    }
  }
</script>

<style scoped lang="less">
</style>
