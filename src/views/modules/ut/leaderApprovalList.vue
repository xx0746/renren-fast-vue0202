<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="月份">
        <el-date-picker v-model="dataForm.showMonth" type="month" value-format="yyyy-MM" :clearable="false"
                        placeholder="选择月份" style="width: 120px" :picker-options="pickerOptions" @change="getDataList()">
        </el-date-picker>
        <!--<el-button type="primary" @click="getDataList()">查询</el-button>-->
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
      <el-table-column label="本月工时" prop="monthut" header-align="center" align="center"></el-table-column>
      <el-table-column label="启动时间" prop="startTime" :formatter="formatColumnDate" header-align="center" width="95"
                       align="center"></el-table-column>
      <el-table-column label="完成时间" prop="endTime" :formatter="formatColumnDate" width="95" header-align="center"
                       align="center"></el-table-column>
      <el-table-column label="牵头部门" prop="departmentName" header-align="center" align="center"
                       width="150"></el-table-column>
      <el-table-column label="项目组长" prop="userName" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="100">
        <!--<template slot-scope="scope">
          <el-button type="text" size="small" @click="updateHandle(scope.row.projectId)">详情</el-button>
        </template>-->
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="updateHandle(scope.row.id,'1')">审核</el-button>
          <span>|</span>
          <el-button type="text" size="small" @click="updateHandle(scope.row.id,'2')">已审核</el-button>
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
    <leader-approval ref="leaderApproval" @refreshDataList="getDataList"></leader-approval>
  </div>
</template>

<script>
  import LeaderApproval from './leaderApproval'

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
        dataList: [],
        detailList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false
      }
    },
    components: {
      LeaderApproval
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
      updateHandle (projectId, queryType) {
        if (this.dataForm.showMonth === '') {
          this.$message.info('请先选择月份')
          return
        }
        let yearMonth = this.dataForm.showMonth.substring(0, 4) + this.dataForm.showMonth.substring(5, 7)
        this.$http({
          url: this.$http.adornUrl('/approval/leaderApprovalWorkTimeList'),
          method: 'post',
          data: this.$http.adornData({
            'yearMonth': yearMonth,
            'projectId': projectId,
            'queryType': queryType
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.detailList = data.dataList
            this.$refs.leaderApproval.init(projectId, this.detailList, queryType, yearMonth)
          }
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/approval/leaderProjectApprovalList'),
          method: 'post',
          data: this.$http.adornData({
            'current': this.pageIndex,
            'size': this.pageSize,
            'yearMonth': this.dataForm.showMonth.substring(0, 4) + this.dataForm.showMonth.substring(5, 7)
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
      }
    }
  }
</script>
