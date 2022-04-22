<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="日">
        <el-date-picker
          v-model="dataForm.dayvalue"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="选择日期">
        </el-date-picker>
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
      <el-table-column label="今日工时" prop="dayut" header-align="center" align="center"></el-table-column>
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
    <manager-day-approval ref="managerDayApproval" @refreshDataList="getDataList"></manager-day-approval>
  </div>
</template>

<script>
  import managerDayApproval from './managerDayApproval'

  export default {
    data () {
      return {
        dataForm: {
          dayvalue: '',
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
      managerDayApproval
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
        let yearMonthDay = this.dataForm.year + '' + this.dataForm.month + '' + this.dataForm.day
        this.$http({
          url: this.$http.adornUrl('/approval/managerDayApprovalWorkTimeList'),
          method: 'post',
          data: this.$http.adornData({
            'yearMonthDay': yearMonthDay,
            'queryType': type,
            'projectId': projectId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.detailList = data.dataList
            this.$refs.managerDayApproval.init(type, projectId, this.dataForm.year, this.dataForm.month, this.dataForm.day, this.detailList)
          } else {
            this.$message.error('程序异常，请联系管理员')
          }
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        let yearMonthDay = this.dataForm.year + '' + this.dataForm.month + '' + this.dataForm.day
        this.$http({
          url: this.$http.adornUrl('/approval/managerProjectDayApprovalList'),
          method: 'post',
          data: this.$http.adornData({
            'current': this.pageIndex,
            'size': this.pageSize,
            'yearMonthDay': yearMonthDay
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
        this.dataForm.year = year
        this.dataForm.month = month
        this.dataForm.day = day
        this.dataForm.dayvalue = year + '-' + month + '-' + day
      }
    }
  }
</script>
