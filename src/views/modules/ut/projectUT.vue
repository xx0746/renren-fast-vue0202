<template>
  <div class="mod-config">

    <el-row>
      <el-button type="primary" @click="dialogVisible = true">主要按钮</el-button>
    </el-row>

    <el-table
      :data="records"
      border
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="year"
        header-align="center"
        align="center"
        label="年份">
      </el-table-column>
      <el-table-column
        prop="projectName"
        header-align="center"
        align="center"
        label="项目名">
      </el-table-column>
      <el-table-column
        prop="projectFrom"
        header-align="center"
        align="center"
        label="项目负责人">
      </el-table-column>

      <el-table-column
        prop="projectCode"
        header-align="center"
        align="center"
        label="项目编码">
      </el-table-column>
      <el-table-column
        prop="projectStatus"
        header-align="center"
        align="center"
        label="项目状态">
      </el-table-column>
      <el-table-column
        prop="allProjectPlanUT"
        header-align="center"
        align="center"
        label="全项目计划UT">
      </el-table-column>
      <el-table-column
        prop="allProjectUsedUT"
        header-align="center"
        align="center"
        label="全项目历年使用UT">
      </el-table-column>
      <el-table-column
        prop="nowYearPlanUT"
        header-align="center"
        align="center"
        label="本年度计划UT">
      </el-table-column>
      <el-table-column
        prop="nowYearUsedUT"
        header-align="center"
        align="center"
        label="本年度已使用UT">
      </el-table-column>
      <el-table-column
        prop="nowYearDissolublePlanUT"
        header-align="center"
        align="center"
        label="本年度可拆分UT">
      </el-table-column>

    <!--   操作列   -->
            <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="150"
              label="操作">
              <template slot-scope="scope">

                <el-button  type="text"  @click="deleteProjectById(scope.row.projectId)">删除</el-button>
                <el-button  type="text"  @click="toUserUT(scope.row.projectId)">项目成员</el-button>

              </template>
            </el-table-column>

    </el-table>
    <el-pagination
      @current-change="currentChangeHandle"
      :current-page="current"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="size"
      :total="total"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>

    <!--对话框-->
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
      >
      <el-radio-group v-model="labelPosition" size="small">
        <el-radio-button label="left">左对齐</el-radio-button>
        <el-radio-button label="right">右对齐</el-radio-button>
        <el-radio-button label="top">顶部对齐</el-radio-button>
      </el-radio-group>
      <div style="margin: 20px;"></div>
      <el-form :label-position="labelPosition" label-width="80px" :model="projectUT">
        <el-form-item label="年份">
          <el-input v-model="projectUT.year"></el-input>
        </el-form-item>
        <el-form-item label="项目名">
          <el-input v-model="projectUT.projectName"></el-input>
        </el-form-item>
        <el-form-item label="项目负责人">
          <el-input v-model="projectUT.projectFrom"></el-input>
        </el-form-item>
        <el-form-item label="项目编码">
          <el-input v-model="projectUT.projectCode"></el-input>
        </el-form-item>
        <el-form-item label="项目状态">
          <el-input v-model="projectUT.projectStatus"></el-input>
        </el-form-item><el-form-item label="全项目计划UT">
        <el-input v-model="projectUT.allProjectPlanUT"></el-input>
      </el-form-item>
        <el-form-item label="历年使用UT">
          <el-input v-model="projectUT.allProjectUsedUT"></el-input>
        </el-form-item>
        <el-form-item label="本年度计划UT">
          <el-input v-model="projectUT.nowYearPlanUT"></el-input>
        </el-form-item>
        <el-form-item label="本年度使用UT">
          <el-input v-model="projectUT.nowYearUsedUT"></el-input>
        </el-form-item>
        <el-form-item label="本年度可拆分UT">
          <el-input v-model="projectUT.nowYearDissolublePlanUT"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogSubmit">确 定</el-button>
  </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      current: 1,
      size: 10,
      total: 0,
      records: [],
      projectUT: {
      },
      dialogVisible: false,
      labelPosition: 'right'
    }
  },
  created () {
    this.getListByPage(1, 10)
  },
  methods: {
    getListByPage (current, size) {
      this.$http({
        url: this.$http.adornUrl('/projectUT/selectByPage/' + current + '/' + size),
        method: 'get'
      }).then((data) => {
        this.current = data.data.current
        this.size = data.data.size
        this.total = data.data.total
        this.records = data.data.records
      })
    },
    currentChangeHandle (current) {
      this.getListByPage(current, this.size)
    },
    saveProject () {
      this.$http({
        url: this.$http.adornUrl('/projectUT/save'),
        method: 'post',
        data: this.projectUT
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '添加成功',
            type: 'success',
            duration: 1500,
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    dialogSubmit () {
      this.saveProject()
      this.dialogVisible = false
      this.getListByPage(1, 10)
    },
    deleteProjectById (projectId) {
      this.$http({
        url: this.$http.adornUrl('/projectUT/deleteById/' + projectId),
        method: 'delete'
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '删除成功',
            type: 'success',
            duration: 1500
          })
          this.getListByPage(1, 10)
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    toUserUT (projectId) {
      this.$router.push({
        path: 'userUT',
        query: {id: projectId}
      })
    }
  }
}
</script>
