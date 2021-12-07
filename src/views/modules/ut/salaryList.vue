<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-date-picker v-model="dataForm.showMonth" type="month" value-format="yyyy-MM" :clearable="false"
                        placeholder="选择月份" style="width: 120px" >
        </el-date-picker>
        <el-select v-model="dataForm.departmentValue"
                   clearable placeholder="请选择部门"
                   style="width: 150px;margin-left: 10px"
        >
          <el-option
            v-for="item in departOptions"
            :key="item.id"
            :label="item.departmentName"
            :value="item.id" >
          </el-option>
        </el-select>
        <el-button type="primary" @click="getDataList()" >查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="updateHandle(0,'','')" >新增</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      :header-cell-style="{background:'#eef1f6',color:'#606266'}">
      <el-table-column label="序号" type="index" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column label="工资月份" header-align="center" align="center" >
        <template slot-scope="scope">
          <span>{{scope.row.year}}-{{scope.row.month}}</span>
        </template>
      </el-table-column>
      <el-table-column label="部门名称" prop="departmentName"  header-align="center" align="center"></el-table-column>
      <el-table-column label="工资总额" prop="salary" header-align="center" align="center"></el-table-column>

      <el-table-column label="操作" header-align="center" align="center" width="50">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="updateHandle(scope.row.id,scope.row.departmentId,scope.row.salary)">修改</el-button>
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
    <!-- 弹窗,  新增-->
    <salary-update ref="salaryUpdate" @refreshDataList="getDataList"></salary-update>
  </div>
</template>

<script>
  import SalaryUpdate from './salaryUpdate'

  export default {
    data () {
      return {
        dataForm: {
          showMonth: '',
          year: '',
          month: ''
        },
        departOptions: [],
        dataList: [],
        optionWorks: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false
      }
    },
    components: {
      SalaryUpdate
    },
    activated () {
      let fullYear = new Date().getFullYear()
      let month = new Date().getMonth() + 1
      if (month < 10) {
        month = '0' + month
      }
      this.dataForm.showMonth = fullYear + '-' + month
      this.getDepartments()
      this.getDataList()
    },
    methods: {
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
          }
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/departmentSalary/salaryList'),
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
      // 新增
      updateHandle (id, departmentId, salary) {
        this.$refs.salaryUpdate.init(id, departmentId, salary, this.dataForm.showMonth, this.departOptions)
      }
    }
  }
</script>
