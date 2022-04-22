<template>
  <div class="performance-list">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-date-picker v-model="dataForm.showMonth" type="month" value-format="yyyy-MM" :clearable="false"
                        placeholder="选择月份" style="width: 120px" :picker-options="pickerOptions">
        </el-date-picker>
        <el-select v-model="dataForm.departmentValue"
                   clearable placeholder="请选择部门"
                   style="width: 150px;margin-left: 10px"
                   :clearable="false"
                   :disabled="disabledflag"
        >
          <el-option
            v-for="item in departOptions"
            :key="item.id"
            :label="item.departmentName"
            :value="item.id" >
          </el-option>
        </el-select>
        <el-button type="primary" @click="getDataList()" >查询</el-button>
        <el-button  type="primary" @click="exportExcel()">导出</el-button>
        <span>{{errStr}}</span>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      :header-cell-style="{background:'#eef1f6',color:'#606266'}">
      <el-table-column  label="序号" type="index" header-align="center"  align="center" width="50"></el-table-column>
      <el-table-column label="员工部门" prop="departmentName" header-align="center" align="center"></el-table-column>
      <el-table-column label="员工姓名" prop="username" header-align="center" align="center"></el-table-column>
      <el-table-column label="本月工时" prop="ut" header-align="center" align="center"></el-table-column>
      <el-table-column label="评价系数" prop="score" header-align="center" align="center"></el-table-column>
      <el-table-column label="工时系数" prop="ratio" header-align="center" align="center"></el-table-column>
      <el-table-column label="绩效标准" prop="salaryStandard" header-align="center" align="center"></el-table-column>
      <el-table-column label="员工绩效" prop="performance" header-align="center" align="center"></el-table-column>
      <el-table-column label="平均能力绩效" prop="agvNengSalary" width="120" header-align="center" align="center"></el-table-column>
      <el-table-column label="平均工时绩效" prop="agvUtSalary" width="120" header-align="center" align="center"></el-table-column>
    </el-table>
    <el-pagination style="width: 90%"
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
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
        showMonth: '',
        departmentValue: ''
      },
      errStr: '',
      disabledflag: true,
      departOptions: [],
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
    // this.getDataList()
    let fullYear = new Date().getFullYear()
    let month = new Date().getMonth() + 1
    if (month < 10) {
      month = '0' + month
    }
    this.dataForm.showMonth = fullYear + '-' + month
    this.getDepartments()
  },
  methods: {
    exportExcel () {
      if (this.dataForm.departmentValue === '' || this.dataForm.departmentValue === undefined){
        this.$message.info('请先选择部门')
        return
      }
      // eslint-disable-next-line no-template-curly-in-string
      window.location.href = this.$http.adornUrl('/dataStatistics/exportPerformanceList?year=' +
        this.dataForm.showMonth.substring(0, 4) +
        '&month=' + this.dataForm.showMonth.substring(5, 7) +
        '&departmentId=' + this.dataForm.departmentValue +
        '&token=' + this.$cookie.get('token'))
    },
    getDepartments () {
      this.$http({
        url: this.$http.adornUrl('/sys/department/list'),
        method: 'post',
        data: this.$http.adornParams({
          'page': 1,
          'limit': 100
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.departOptions = data.dataList
          if (this.departOptions.length > 1) {
            this.disabledflag = false
          } else if (this.departOptions.length === 1) {
            this.dataForm.departmentValue = this.departOptions[0].id
            this.disabledflag = true
          } else {
            this.disabledflag = true
          }
        } else {
          this.disabledflag = true
        }
      })
    },
    // 获取数据列表
    getDataList () {
      if (this.dataForm.departmentValue === '' || this.dataForm.departmentValue === undefined){
        this.$message.info('请先选择部门')
        return
      }
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/dataStatistics/performanceList'),
        method: 'post',
        data: this.$http.adornData({
          'current': this.pageIndex,
          'size': this.pageSize,
          'year': this.dataForm.showMonth.substring(0, 4),
          'month': this.dataForm.showMonth.substring(5, 7),
          'departmentId': this.dataForm.departmentValue
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.dataList
          this.totalPage = data.total
          this.errStr = data.errStr
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    }
  }
}
</script>

<style scoped>

</style>
