<template>
  <div class="performance-score">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item  label="月份">
        <el-date-picker v-model="dataForm.showMonth" type="month" value-format="yyyy-MM" :clearable="false"
                        placeholder="选择月份" style="width: 120px" :picker-options="pickerOptions" @change="getDataList()">
        </el-date-picker>
        <!--<el-button type="primary" @click="getDataList()" >查询</el-button>-->
        <el-button type="primary" @click="submitDataList()" v-if="!disabledflag">提交</el-button>
        <span v-if="!disabledflag" style="font-size: 1px; color: #999999">温馨提示：所有人打完分以后，请点击提交，每个月只能提交一次。</span>
        <span v-if="disabledflag" style="font-size: 1px; color: #999999">温馨提示：本月已打分，每个月只能提交一次。</span>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      :header-cell-style="{background:'#eef1f6',color:'#606266'}">
      <el-table-column  label="序号" type="index" header-align="center" fixed align="center" width="50"></el-table-column>
      <el-table-column label="月份" header-align="center" align="center" >
        <template slot-scope="scope">
          <span>{{scope.row.year}}-{{scope.row.month}}</span>
        </template>
      </el-table-column>
      <el-table-column label="部门" prop="departmentName" header-align="center" align="center"></el-table-column>
      <el-table-column label="姓名" prop="username" header-align="center" align="center"></el-table-column>
     <!-- <el-table-column  label="分数" prop="score" header-align="center" align="center"></el-table-column>-->
      <el-table-column  label="分数" header-align="center" align="center">
        <template slot-scope="scope">
          <el-input-number v-if="!disabledflag" v-model="scope.row.score" :precision="2" :step="1" >
          </el-input-number>
          <span v-if="disabledflag">{{scope.row.score}}</span>
        </template>
      </el-table-column>
    </el-table>

  </div>
</template>

<script>
export default {
  data () {
    return {
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() > Date.now()
        }
      },
      dataForm: {
        showMonth: ''
      },
      disabledflag: true,
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false
    }
  },
  components: {
  },
  activated () {
    let fullYear = new Date().getFullYear()
    let month = new Date().getMonth() + 1
    if (month < 10) {
      month = '0' + month
    }
    this.dataForm.showMonth = fullYear + '-' + month
    this.getDataList()
  },
  methods: {

    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/userScore/getUserScoreList'),
        method: 'post',
        data: this.$http.adornData({
          'year': this.dataForm.showMonth.substring(0, 4),
          'month': this.dataForm.showMonth.substring(5, 7)
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.dataList
          this.totalPage = data.total
          if (data.isScore === '0') {
            this.disabledflag = false
          } else {
            this.disabledflag = true
          }
        } else {
          this.dataList = []
          this.totalPage = 0
          this.disabledflag = true
        }
        this.dataListLoading = false
      })
    },
    submitDataList () {
      this.$confirm(`提交后无法修改，确定提交吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/userScore/saveUserScore'),
          method: 'post',
          data: this.$http.adornData({
            'dataList': this.dataList
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.getDataList()
          }
          this.dataListLoading = false
        })
      })
    }
  }
}
</script>
