<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="500px"
    v-loading="dataListLoading"
  >
    <el-table
    :data="detailList"
    border
    style="width: 100%;">
    <el-table-column  label="序号" type="index" header-align="center" align="center" width="50"></el-table-column>
    <el-table-column label="员工" prop="userName" header-align="center" align="center"></el-table-column>
    <el-table-column label="工时" prop="monthTime" header-align="center" align="center"></el-table-column>
  </el-table>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="visible = false">关闭</el-button>
      <el-button type="primary" @click="dataFormSubmit()" v-if="submitFlag">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        submitFlag: false,
        detailList: [],
        dataListLoading: false,
        dataForm: {
          projectId: '',
          yearMonth: ''
        }
      }
    },
    methods: {
      init (projectId, detailList, queryType, yearMonth) {
        this.dataForm.projectId = projectId
        this.dataForm.yearMonth = yearMonth
        this.detailList = detailList
        this.visible = true
        if (this.detailList.length === 0) {
          this.submitFlag = false
        } else {
          if (queryType === '1') {
            this.submitFlag = true
          } else {
            this.submitFlag = false
          }
        }
      },
      dataFormSubmit () {
        if (this.dataListLoading === true) {
          this.$message.info('审批中，请稍等')
          return
        }
        if (this.detailList.length <= 0) {
          this.$message.info('没有待审批工时')
          return
        }
        this.$confirm(`确定审批吗?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.dataListLoading = true
          this.$http({
            url: this.$http.adornUrl(`/approval/leaderApprovalWorkTime`),
            method: 'post',
            data: this.$http.adornData({
              'projectId': this.dataForm.projectId,
              'yearmonth': this.dataForm.yearMonth
            })
          }).then(({data}) => {
            this.dataListLoading = false
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      }
    }
  }
</script>
<style>
  .el-col {
    border-radius: 4px;
  }

  .bg-purple {
    background: #d3dce6;
  }

  .grid-content {
    border-radius: 4px;
    min-height: 20px;
  }
  .row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
  }
</style>
