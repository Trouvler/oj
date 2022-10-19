<template>
  <div>
    <el-input
      v-model="keyword"
      placeholder="Keywords"
      prefix-icon="el-icon-search">
    </el-input>
    <el-table :data="quizs" v-loading="loading">
      <el-table-column
        label="ID"
        width="100"
        prop="id">
      </el-table-column>
      <el-table-column
        label="DisplayID"
        width="200"
        prop="_id">
      </el-table-column>
      <el-table-column
        label="Title"
        prop="title">
      </el-table-column>
      <el-table-column
        label="option"
        align="center"
        width="100"
        fixed="right">
        <template slot-scope="{row}">
          <icon-btn icon="plus" name="Add the quiz"
                    @click.native="handleAddQuiz(row.id)"></icon-btn>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="page"
      layout="prev, pager, next"
      @current-change="getPublicQuiz"
      :page-size="limit"
      :total="total">
    </el-pagination>
  </div>
</template>
<script>
  import api from '@admin/api'

  export default {
    name: 'add-quiz-from-public',
    props: ['contestID'],
    data () {
      return {
        page: 1,
        limit: 10,
        total: 0,
        loading: false,
        quizs: [],
        contest: {},
        keyword: ''
      }
    },
    mounted () {
      api.getContest(this.contestID).then(res => {
        this.contest = res.data.data
        this.getPublicQuiz()
      }).catch(() => {
      })
    },
    methods: {
      getPublicQuiz (page) {
        this.loading = true
        let params = {
          keyword: this.keyword,
          offset: (page - 1) * this.limit,
          limit: this.limit,
          rule_type: this.contest.rule_type
        }
        api.getQuizList(params).then(res => {
          this.loading = false
          this.total = res.data.data.total
          this.quizs = res.data.data.results
        }).catch(() => {
        })
      },
      handleAddQuiz (quizID) {
        this.$prompt('Please input display id for the contest quiz', 'confirm').then(({value}) => {
          let data = {
            quiz_id: quizID,
            contest_id: this.contestID,
            display_id: value
          }
          api.addQuizFromPublic(data).then(() => {
            this.$emit('on-change')
          }, () => {
          })
        }, () => {
        })
      }
    },
    watch: {
      'keyword' () {
        this.getPublicQuiz(this.page)
      }
    }
  }
</script>
<style scoped>
  .page {
    margin-top: 20px;
    text-align: right
  }

</style>
