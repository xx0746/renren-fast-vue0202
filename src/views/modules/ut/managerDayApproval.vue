<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="700px"
  >

    <el-form :model="dataForm"  ref="dataForm" @keyup.enter.native="dataFormSubmit()" v-loading="dataListLoading" >
      <el-table
        :data="detailList"
        border
        style="width: 100%;">
        <el-table-column label="序号" type="index" header-align="center"  align="center" width="50"></el-table-column>
        <el-table-column label="员工" prop="username" header-align="center"  width="100" align="center"></el-table-column>
        <el-table-column label="今日总工时" prop="dayTime" header-align="center"  width="100" align="center" ></el-table-column>
        <el-table-column label="工作明细"  header-align="center" align="center" >
          <template slot-scope="scope">
            <el-row v-for="(detailgongshi,index1) in scope.row.workTypelist" :key="'aaaa'+index1"
                    style="margin-top: 3px;">
              <el-col :span="18" style="text-align: left">
                <div class="grid-content ">工作{{index1+1}}：{{detailgongshi.oneName}}-{{detailgongshi.twoName}}</div>
              </el-col>
              <el-col :span="6">
                <!--<el-input v-model="detailgongshi.ut" placeholder="工时" :disabled="disabledflag"
                          style="width: 50px; text-align: center" size="mini" @change="chekNum($event)">
                </el-input>-->
                <el-input-number :disabled="disabledflag" v-model="detailgongshi.ut" :precision="1" :step="1"
                                 :controls="false" style="width: 50px; text-align: center" size="mini">
                </el-input-number>
                工时
              </el-col>
            </el-row >
          </template>
        </el-table-column>
      </el-table>
    </el-form>
    <span slot="footer" class="dialog-footer" >
      <el-button @click="visible = false" type="primary">关闭</el-button>
      <el-button v-if="visibleOK" type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        visibleOK: false,
        disabledflag: true,
        visibleRow: true,
        detailList: [],
        dataListLoading: false,
        dataForm: {
          projectId: '',
          year: '',
          month: '',
          week: ''
        }
      }
    },
    methods: {
      init (type, projectId, year, month, week, detailList) {
        this.dataForm.projectId = projectId
        this.dataForm.year = year
        this.dataForm.month = month
        this.dataForm.day = week
        this.detailList = detailList
        this.visible = true
        if (type === '2') {
          this.disabledflag = false
          if (this.detailList.length > 0) {
            this.visibleOK = true
            this.visibleRow = false
          } else {
            this.visibleOK = false
            this.visibleRow = true
          }
        } else {
          this.visibleOK = false
          this.visibleRow = false
          this.disabledflag = true
        }
      },
      // 表单提交
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
            url: this.$http.adornUrl(`/approval/managerDayApprovalWorkTime`),
            method: 'post',
            data: this.$http.adornData({
             /* 'projectId': this.dataForm.projectId,
              'yearMonthWeek': this.dataForm.year + this.dataForm.month + this.dataForm.day, */
              'dataList': this.detailList
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
