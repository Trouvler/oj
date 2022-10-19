<template>
  <div class="view">
    <Panel :title="contestId ? this.$i18n.t('m.Contest_Quiz_List') : this.$i18n.t('m.Quiz_List')">
      <div slot="header">
        <el-input
          v-model="keyword"
          prefix-icon="el-icon-search"
          placeholder="Keywords">
        </el-input>
      </div>
      <el-table
        v-loading="loading"
        element-loading-text="loading"
        ref="table"
        :data="zList"
        @row-dblclick="handleDblclick"
        style="width: 100%">
        <el-table-column
          width="100"
          prop="id"
          label="ID">
        </el-table-column>
        <el-table-column
          width="150"
          label="Display ID">
          <template slot-scope="{row}">
            <span v-show="!row.isEditing">{{row._id}}</span>
            <el-input v-show="row.isEditing" v-model="row._id"
                      @keyup.enter.native="handleInlineEdit(row)">

            </el-input>
          </template>
        </el-table-column>
        <el-table-column
          prop="title"
          label="Title">
          <template slot-scope="{row}">
            <span v-show="!row.isEditing">{{row.title}}</span>
            <el-input v-show="row.isEditing" v-model="row.title"
                      @keyup.enter.native="handleInlineEdit(row)">
            </el-input>
          </template>
        </el-table-column>
        <el-table-column
          prop="created_by.username"
          label="Author">
        </el-table-column>
        <el-table-column
          width="200"
          prop="create_time"
          label="Create Time">
          <template slot-scope="scope">
            {{scope.row.create_time | localtime }}
          </template>
        </el-table-column>
        <el-table-column
          width="100"
          prop="visible"
          label="Visible">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.visible"
                       active-text=""
                       inactive-text=""
                       @change="updateQuiz(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          fixed="right"
          label="Operation"
          width="250">
          <div slot-scope="scope">
            <icon-btn name="Edit" icon="edit" @click.native="goEdit(scope.row.id)"></icon-btn>
            <icon-btn v-if="contestId" name="Make Public" icon="clone"
                      @click.native="makeContestQuizPublic(scope.row.id)"></icon-btn>
            <icon-btn icon="download" name="Download TestCase"
                      @click.native="downloadTestCase(scope.row.id)"></icon-btn>
            <icon-btn icon="trash" name="Delete Quiz"
                      @click.native="deleteQuiz(scope.row.id)"></icon-btn>
          </div>
        </el-table-column>
      </el-table>
      <div class="panel-options">
        <el-button type="primary" size="small"
                   @click="goCreateQuiz" icon="el-icon-plus">Create
        </el-button>
        <el-button v-if="contestId" type="primary"
                   size="small" icon="el-icon-plus"
                   @click="addQuizDialogVisible = true">Add From Public Quiz
        </el-button>
        <el-pagination
          class="page"
          layout="prev, pager, next"
          @current-change="currentChange"
          :page-size="pageSize"
          :total="total">
        </el-pagination>
      </div>
    </Panel>
    <el-dialog title="Sure to update the quiz? "
               width="20%"
               :visible.sync="InlineEditDialogVisible"
               @close-on-click-modal="false">
      <div>
        <p>DisplayID: {{currentRow._id}}</p>
        <p>Title: {{currentRow.title}}</p>
      </div>
      <span slot="footer">
        <cancel @click.native="InlineEditDialogVisible = false; getQuizList(currentPage)"></cancel>
        <save @click.native="updateQuiz(currentRow)"></save>
      </span>
    </el-dialog>
    <el-dialog title="Add Contest Quiz"
               v-if="contestId"
               width="80%"
               :visible.sync="addQuizDialogVisible"
               @close-on-click-modal="false">
      <add-quiz-component :contestID="contestId" @on-change="getQuizList"></add-quiz-component>
    </el-dialog>
  </div>
</template>

<script>
  import api from '../../api.js'
  import utils from '@/utils/utils'
  import AddQuizComponent from './AddPublicQuiz.vue'

  export default {
    name: 'QuizList',
    components: {
      AddQuizComponent
    },
    data () {
      return {
        pageSize: 10,
        total: 0,
        quizList: [],
        keyword: '',
        loading: false,
        currentPage: 1,
        routeName: '',
        contestId: '',
        // for make public use
        currentQuizID: '',
        currentRow: {},
        InlineEditDialogVisible: false,
        makePublicDialogVisible: false,
        addQuizDialogVisible: false
      }
    },
    mounted () {
      this.routeName = this.$route.name
      this.contestId = this.$route.params.contestId
      this.getQuizList(this.currentPage)
    },
    methods: {
      handleDblclick (row) {
        row.isEditing = true
      },
      goEdit (quizId) {
        if (this.routeName === 'quiz-list') {
          this.$router.push({name: 'edit-quiz', params: {quizId}})
        } else if (this.routeName === 'contest-quiz-list') {
          this.$router.push({name: 'edit-contest-quiz', params: {quizId: quizId, contestId: this.contestId}})
        }
      },
      goCreateQuiz () {
        if (this.routeName === 'quiz-list') {
          this.$router.push({name: 'create-quiz'})
        } else if (this.routeName === 'contest-quiz-list') {
          this.$router.push({name: 'create-contest-quiz', params: {contestId: this.contestId}})
        }
      },
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getQuizList(page)
      },
      getQuizList (page = 1) {
        this.loading = true
        let funcName = this.routeName === 'quiz-list' ? 'getQuizList' : 'getContestQuizList'
        let params = {
          limit: this.pageSize,
          offset: (page - 1) * this.pageSize,
          keyword: this.keyword,
          contest_id: this.contestId
        }
        api[funcName](params).then(res => {
          this.loading = false
          this.total = res.data.data.total
          for (let quiz of res.data.data.results) {
            quiz.isEditing = false
          }
          this.quizList = res.data.data.results
        }, res => {
          this.loading = false
        })
      },
      deleteQuiz (id) {
        this.$confirm('Sure to delete this quiz? The associated submissions will be deleted as well.', 'Delete Quiz', {
          type: 'warning'
        }).then(() => {
          let funcName = this.routeName === 'quiz-list' ? 'deleteQuiz' : 'deleteContestQuiz'
          api[funcName](id).then(() => [
            this.getQuizList(this.currentPage - 1)
          ]).catch(() => {
          })
        }, () => {
        })
      },
      makeContestQuizPublic (quizID) {
        this.$prompt('Please input display id for the public quiz', 'confirm').then(({value}) => {
          api.makeContestQuizPublic({id: quizID, display_id: value}).catch()
        }, () => {
        })
      },
      updateQuiz (row) {
        let data = Object.assign({}, row)
        let funcName = ''
        if (this.contestId) {
          data.contest_id = this.contestId
          funcName = 'editContestQuiz'
        } else {
          funcName = 'editQuiz'
        }
        api[funcName](data).then(res => {
          this.InlineEditDialogVisible = false
          this.getQuizList(this.currentPage)
        }).catch(() => {
          this.InlineEditDialogVisible = false
        })
      },
      handleInlineEdit (row) {
        this.currentRow = row
        this.InlineEditDialogVisible = true
      },
      downloadTestCase (quizID) {
        let url = '/admin/test_case?quiz_id=' + quizID
        utils.downloadFile(url)
      },
      getPublicQuiz () {
        api.getQuizList()
      }
    },
    watch: {
      '$route' (newVal, oldVal) {
        this.contestId = newVal.params.contestId
        this.routeName = newVal.name
        this.getQuizList(this.currentPage)
      },
      'keyword' () {
        this.currentChange()
      }
    }
  }
</script>

<style scoped lang="less">
</style>
