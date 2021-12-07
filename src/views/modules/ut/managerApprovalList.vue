<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="周">
        <el-date-picker
          style="width: 175px"
          :clearable="false"
          v-model="dataForm.weekvalue"
          :format="dataForm.year +'-' +dataForm.month +'月第' + dataForm.week + '周'"
          @change="weekChange"
          value-format="yyyy-MM-dd"
          class="inp"
          size="medium"
          type="week"
          placeholder="请选择"
          :picker-options="{'firstDayOfWeek': 1}"
          end-placeholder
        ></el-date-picker>
        <!--<el-button type="primary" @click="getDataList()">查询</el-button>-->
        <span>({{this.weekStart}}日至{{this.weekEnd}}日)</span>
      </el-form-item>

    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      :header-cell-style="{background:'#eef1f6',color:'#606266'}">
      <el-table-column label="序号" type="index" header-align="center" fixed align="center" width="50"></el-table-column>
      <el-table-column label="项目编号" prop="code" header-align="center" align="center" width="100"></el-table-column>
      <el-table-column label="项目名称" prop="name" header-align="center" align="center" width="250"></el-table-column>
      <el-table-column label="总工时" prop="ut" header-align="center" align="center"></el-table-column>
      <el-table-column label="已用工时" prop="usedut" header-align="center" align="center"></el-table-column>
      <el-table-column label="剩余工时" prop="unusedut" header-align="center" align="center"></el-table-column>
      <el-table-column label="本周工时" prop="weekut" header-align="center" align="center"></el-table-column>
      <el-table-column label="启动时间" prop="startTime" :formatter="formatColumnDate" header-align="center" width="95"
                       align="center"></el-table-column>
      <el-table-column label="完成时间" prop="endTime" :formatter="formatColumnDate" width="95" header-align="center"
                       align="center"></el-table-column>
      <el-table-column label="牵头部门" prop="departmentName" header-align="center" align="center"
                       width="150"></el-table-column>
      <el-table-column label="项目组长" prop="userName" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="100">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="updateHandle(scope.row.id,'2')">审核</el-button>
          <span>|</span>
          <el-button type="text" size="small" @click="updateHandle(scope.row.id,'1')">已审核</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <manager-approval ref="managerApproval" @refreshDataList="getDataList"></manager-approval>
  </div>
</template>

<script>
  import ManagerApproval from './managerApproval'

  export default {
    data () {
      return {
        dataForm: {
          weekvalue: '',
          year: '',
          month: '',
          week: ''
        },
        weekStart: '',
        weekEnd: '',
        dataList: [],
        detailList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false
      }
    },
    components: {
      ManagerApproval
    },
    activated () {
      this.getDay()
      this.getDataList()
    },
    methods: {
      formatColumnDate (row, column) {
        // 获取单元格数据
        let data = row[column.property]
        if (data === 'undefined' || data === '') {
          return ''
        }
        let dt = new Date(data)
        // eslint-disable-next-line no-unused-vars
        let month = dt.getMonth() + 1
        let day = dt.getDate()
        if (month < 10) {
          month = '0' + month
        }
        if (day < 10) {
          day = '0' + day
        }
        return dt.getFullYear() + '-' + month + '-' + day
      },
      updateHandle (projectId, type) {
        this.$http({
          url: this.$http.adornUrl('/approval/managerApprovalWorkTimeList'),
          method: 'post',
          data: this.$http.adornData({
            'yearMonthWeek': this.dataForm.year + this.dataForm.month + this.dataForm.week,
            'queryType': type,
            'projectId': projectId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.detailList = data.dataList
            this.$refs.managerApproval.init(type, projectId, this.dataForm.year, this.dataForm.month, this.dataForm.week, this.detailList)
          } else {
            this.$message.error('程序异常，请联系管理员')
          }
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/approval/managerProjectApprovalList'),
          method: 'post',
          data: this.$http.adornData({
            'current': this.pageIndex,
            'size': this.pageSize,
            'yearMonthWeek': this.dataForm.year + this.dataForm.month + this.dataForm.week
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.dataList
            this.totalPage = data.total
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
      },
      weekChange (val) {
        if (val) {
          this.dataForm.week = this.getWeekInMonth(new Date(val))
          this.getDataList()
        }
      },
      getDay () {
        let date = new Date()
        let year = date.getFullYear()
        let month = date.getMonth() + 1
        let day = date.getDate()
        if (month < 10) {
          month = '0' + month
        }
        if (day < 10) {
          day = '0' + day
        }
        let nowDay = year + '-' + month + '-' + day
        this.dataForm.week = this.getWeekInMonth(new Date(nowDay))
        this.dataForm.weekvalue = year + '-' + month + '-' + day
      },

      // 根据日期判断是月的第几周
      getWeekInMonth (t) {
        if (t === undefined || t === '' || t === null) {
          t = new Date()
        } else {
          let now = new Date(t)
          let nowTime = now.getTime()
          let day = now.getDay() || 7
          let oneDayTime = 24 * 60 * 60 * 1000
          let MondayTime = nowTime - (day - 1) * oneDayTime// 显示周一
          let SundayTime = nowTime + (7 - day) * oneDayTime// 显示周日
          let weekStartDate = new Date(MondayTime)
          this.weekStart = this.formatDate(new Date(MondayTime))
          this.weekEnd = this.formatDate(new Date(SundayTime))
          this.dataForm.year = weekStartDate.getFullYear() + ''
          let month = weekStartDate.getMonth() + 1
          if (month < 10) {
            month = '0' + month
          }
          this.dataForm.month = month + ''
          let date = weekStartDate.getDate()
          // eslint-disable-next-line no-unused-vars
          weekStartDate.setDate(1)
          let d = weekStartDate.getDay()
          let fisrtWeekend = d
          if (d === 0) {
            fisrtWeekend = 1
          } else {
            fisrtWeekend = 7 - d + 1
          }
          if (date <= fisrtWeekend) {
            return '1'
          } else {
            return 1 + Math.ceil((date - fisrtWeekend) / 7) + ''
          }
        }
      },
      formatDate (date) {
        let myyear = date.getFullYear()
        let mymonth = date.getMonth() + 1
        let myweekday = date.getDate()
        if (mymonth < 10) {
          mymonth = '0' + mymonth
        }
        if (myweekday < 10) {
          myweekday = '0' + myweekday
        }
        return myyear + '-' + mymonth + '-' + myweekday
      }
    }
  }
</script>
