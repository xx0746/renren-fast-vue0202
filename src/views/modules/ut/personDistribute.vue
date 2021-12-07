<template>
  <div class="mod-user">
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      :header-cell-style="{background:'#eef1f6',color:'#606266'}">
      <el-table-column  label="序号" type="index" header-align="center" fixed align="center" width="50"></el-table-column>
      <el-table-column label="项目编号" prop="code" header-align="center" align="center"  width="100"></el-table-column>
      <el-table-column label="项目名称" prop="name" header-align="center" align="center" width="250"></el-table-column>
      <el-table-column label="启动时间" prop="startTime" :formatter="formatDate" header-align="center"
                       width="95" align="center"></el-table-column>
      <el-table-column label="完成时间" prop="endTime"  :formatter="formatDate" width="95" header-align="center" align="center"></el-table-column>
      <el-table-column label="牵头部门" prop="departmentName" header-align="center"  align="center"  width="150"></el-table-column>
      <el-table-column label="项目组长" prop="userName" header-align="center" align="center"></el-table-column>
      <el-table-column label="总工时" prop="ut" header-align="center" align="center"></el-table-column>
      <el-table-column label="立项依据" prop="according" header-align="center" align="center"></el-table-column>
      <el-table-column label="项目级别" prop="level" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="50">
        <template slot-scope="scope">
        <el-button  type="text" size="small" @click="updateHandle(scope.row.id,scope.row.code,scope.row.name,scope.row.ut)">分配</el-button>
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
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
        },
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
      this.getDataList()
    },
    methods: {
      formatDate (row, column) {
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
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/project/projectList'),
          method: 'post',
          data: this.$http.adornData({
            'current': this.pageIndex,
            'size': this.pageSize,
            'queryType': '1'
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.dataList
            this.totalPage = data.total
          } else {
            this.dataList = []
            this.totalPage = 0
            this.$message.error(data.msg)
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
      // 分配人员
      updateHandle (id, code, name, ut) {
        this.$router.push({'name': 'personList', params: {id: id, code: code, name: name, ut: ut}})
      }
    }
  }
</script>
