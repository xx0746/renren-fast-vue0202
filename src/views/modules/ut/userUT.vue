。<template>
  <div class="mod-config">
    <div class="floatLeft">
      <el-row>
        <el-button type="primary" @click="openDialog">添加员工</el-button>
      </el-row>
    </div>

    <div class="floatLeft">
      <div class="block">
        <span class="demonstration">月</span>
        <el-date-picker
          value-format="yyyy-MM"
          format="yyyy-MM"
          v-model="userUT.date"
          type="month"
          placeholder="选择月"
        @change="dateChange">
        </el-date-picker>
      </div>
    </div>


    <el-table
      :data="records"
      border
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="projectName"
        header-align="center"
        align="center"
        label="项目名">
      </el-table-column>
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="用户名">
      </el-table-column>
      <el-table-column
        prop="planUT"
        header-align="center"
        align="center"
        label="全项目计划UT">
      </el-table-column>

      <el-table-column
        prop="usedUT"
        header-align="center"
        align="center"
        label="历年使用UT">
      </el-table-column>
      <el-table-column
        prop="nowMonthPlanUT"
        header-align="center"
        align="center"
        label="本月计划UT">
      </el-table-column>
      <el-table-column
        prop="nowMonthUsedUT"
        header-align="center"
        align="center"
        label="本月已使用UT">
      </el-table-column>
      <el-table-column
        prop="nowMonthAllUT"
        header-align="center"
        align="center"
        label="本月所有项目已使用UT">
      </el-table-column>

      <!--   操作列   -->
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">

          <el-button  type="text"  @click="deleteProjectById(scope.row.projectId)">删除</el-button>
          <el-button  type="text"  @click="toFillUT(scope.row.projectId,scope.row.userId)">填写UT</el-button>

        </template>
      </el-table-column>

    </el-table>
    <el-pagination
      @current-change="currentChangeHandle"
      :current-page="current"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="size"
      :total="total"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>

    <!--  对话框  -->
    <!--对话框-->
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <div style="margin: 20px;"></div>

      <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选</el-checkbox>
      <div style="margin: 15px 0;"></div>
      <el-checkbox-group v-model="checkedUsers" @change="handleCheckedCitiesChange">
        <el-checkbox v-for="user in allUsers" :label="user.userId" :key="user.userId">{{user.username}}</el-checkbox>
      </el-checkbox-group>

      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogSubmit">确 定</el-button>
  </span>
    </el-dialog>

    <!--  第二个对话框  -->
    <el-dialog
      title="提示"
      :visible.sync="twoDialogVisible"
      width="30%"
    >
      <div style="margin: 20px;"></div>

      <el-radio-group v-model="labelPosition" size="small">
        <el-radio-button label="left">左对齐</el-radio-button>
        <el-radio-button label="right">右对齐</el-radio-button>
        <el-radio-button label="top">顶部对齐</el-radio-button>
      </el-radio-group>
      <div style="margin: 20px;"></div>
      <el-form :label-position="labelPosition" label-width="80px" :model="userUT">
        <el-form-item label="全项目计划UT">
          <el-input v-model="userUT.planUT"></el-input>
        </el-form-item>
        <el-form-item label="本年月计划UT">
          <el-input v-model="userUT.nowMonthPlanUT"></el-input>
        </el-form-item>
        <el-form-item label="本月使用UT">
          <el-input v-model="userUT.nowMonthUsedUT"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
    <el-button @click="twoDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="fillOrUpdate">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      current: 1,
      size: 10,
      total: 0,
      records: [],
      dialogVisible: false,
      checkedUsers: [],
      allUsers: [],
      checkAll: false,
      isIndeterminate: true,
      userUT: {},
      twoDialogVisible: false,
      labelPosition: 'left'
    }
  },
  created () {
    this.pageUserUTWithProjectId(this.$route.query.id, 1, 10)
  },
  watch: {
    $route () {
      if (this.$route.path == '/userUT') {
        this.pageUserUTWithProjectId(this.$route.query.id, 1, 10)
      }
    }
  },
  methods: {
    pageUserUTWithProjectId (projectId, current, size) {
      this.$http({
        url: this.$http.adornUrl('/userUT/pageUserUTWithProjectId/' + projectId + '/' + current + '/' + size + '/' + this.userUT.date),
        method: 'get'
      }).then(({data}) => {
        this.current = data.current
        this.size = data.size
        this.total = data.total
        this.records = data.records
      })
    },
    handleCheckAllChange (val) {
      this.checkedUsers = val ? this.allUsers : []
      this.isIndeterminate = false
    },
    handleCheckedCitiesChange (value) {
      let checkedCount = value.length
      this.checkAll = checkedCount === this.allUsers.length
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.allUsers.length
    },
    openDialog () {
      this.dialogVisible = true
      this.$http({
        url: this.$http.adornUrl('/sys/user/list'),
        method: 'get'
      }).then(({data}) => {
        this.allUsers = data.page.list
      })
      this.$http({
        url: this.$http.adornUrl('/projectUT/getUserIds/' + this.$route.query.id + '/' + this.userUT.date),
        method: 'get'
      }).then(({data}) => {
        console.log(data.userIds)
        this.checkedUsers = data.userIds
      })
    },
    dialogSubmit () {
      this.dialogVisible = true
      this.addUserToProject()
    },
    addUserToProject () {
      this.$http({
        url: this.$http.adornUrl('/projectUT/addUserToProject/' + this.$route.query.id + '/' + this.userUT.date),
        method: 'post',
        data: this.checkedUsers
      }).then(({data}) => {
        this.pageUserUTWithProjectId(this.$route.query.id, 1, this.size)
        this.dialogVisible = false
      })
    },
    currentChangeHandle (current) {
      this.pageUserUTWithProjectId(this.$route.query.id, current, this.size)
    },
    toFillUT (projectId, userId) {
      this.getUserUTByProjectIdAndUserId(projectId, userId)
      if (this.userUT.id == null) {
        this.userUT.projectId = projectId
        this.userUT.userId = userId
      }
      this.twoDialogVisible = true
    },
    getUserUTByProjectIdAndUserId (projectId, userId) {
      this.$http({
        url: this.$http.adornUrl('/userUT/getUserUTByProjectIdAndUserId/' + projectId + '/' + userId + '/' + this.userUT.date),
        method: 'get'
      }).then(({data}) => {
        if (data.userUT != null) {
          this.userUT = data.userUT
        }
      })
    },
    fillUT () {
      this.$http({
        url: this.$http.adornUrl('/userUT/fillUT'),
        method: 'post',
        data: this.userUT
      }).then(({data}) => {
        this.userUT = {'date': this.userUT.date}
        this.pageUserUTWithProjectId(this.$route.query.id, 1, 10)
        this.twoDialogVisible = false
      })
    },
    updateUT () {
      this.$http({
        url: this.$http.adornUrl('/userUT/updateUT'),
        method: 'put',
        data: this.userUT
      }).then(({data}) => {
        this.userUT = {'date': this.userUT.date}
        this.pageUserUTWithProjectId(this.$route.query.id, 1, 10)
        this.twoDialogVisible = false
      })
    },
    fillOrUpdate () {
      if (this.userUT.id == null) {
        this.fillUT()
      } else {
        this.updateUT()
      }
    },
    dateChange () {
      this.pageUserUTWithProjectId(this.$route.query.id, 1, 10)
    }
  }
}
</script>

<style lang="css" scoped>
  .floatLeft {
    float: left;
    margin-right: 40px;
  }
</style>
