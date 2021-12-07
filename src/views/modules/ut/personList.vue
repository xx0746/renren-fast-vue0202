<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item  v-if="visible">
        <el-button type="primary"  @click="addHandle()">新增</el-button>
      </el-form-item>
      <el-form-item >
        <span>项目总工时: {{ut}} ; 已分配工时: {{dataForm.usedUt}} ; 剩余工时: {{ut-dataForm.usedUt}}</span>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      :header-cell-style="{background:'#eef1f6',color:'#606266'}">
      <el-table-column  label="序号" type="index" header-align="center"  align="center" width="50"></el-table-column>
      <el-table-column label="员工姓名" prop="name" header-align="center" align="center"></el-table-column>
      <el-table-column label="分配工时" prop="ut" header-align="center" align="center"></el-table-column>
      <el-table-column  label="项目编码"   header-align="center"  align="center">
        <template slot-scope="scope">{{projectCode}}</template>
      </el-table-column>
      <el-table-column  label="项目名称" header-align="center"  align="center" >
        <template slot-scope="scope">{{projectName}}</template>
      </el-table-column>
      <el-table-column label="操作"  header-align="center" align="center" width="50">
        <template slot-scope="scope">
        <el-button  type="text" size="small" @click="deleteHandle(scope.row.userId)">删除</el-button>
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
    <person-add ref="personAdd" @refreshDataList="getDataList"></person-add>
  </div>
</template>

<script>
  import PersonAdd from './personAdd'
  export default {
    data () {
      return {
        dataForm: {
          usedUt: 0
        },
        visible: false,
        projectId: '',
        projectCode: '',
        projectName: '',
        ut: 0,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false
      }
    },
    components: {
      PersonAdd
    },
    activated () {
      this.projectId = this.$route.params.id
      this.projectCode = this.$route.params.code
      this.projectName = this.$route.params.name
      this.ut = this.$route.params.ut
      this.getDataList()
    },
    methods: {

      // 获取数据列表
      getDataList () {
        this.visible = false
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/userProject/personList'),
          method: 'post',
          params: this.$http.adornParams({
            'current': this.pageIndex,
            'size': this.pageSize,
            'projectId': this.projectId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.dataList
            this.totalPage = data.total
            this.dataForm.usedUt = data.usedUt
            this.visible = true
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
      deleteHandle (userId) {
        this.$confirm(`确定删除吗?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl(`/userProject/deletePerson`),
            method: 'post',
            params: this.$http.adornParams({
              'personId': userId,
              'projectId': this.projectId
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      },
      // 新增
      addHandle () {
        let unuserut = this.ut - this.dataForm.usedUt
        this.$refs.personAdd.init(this.projectId, unuserut)
      }
    }
  }
</script>
