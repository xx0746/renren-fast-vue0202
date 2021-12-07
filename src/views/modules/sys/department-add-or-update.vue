<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="部门名称" prop="departmentName">
        <el-input v-model="dataForm.departmentName" placeholder="部门名称"></el-input>
      </el-form-item>

      <!--   <el-form-item label="部门" size="mini" prop="roleIdList">
           <el-checkbox-group v-model="dataForm.roleIdList">
             <el-checkbox v-for="role in roleList" :key="role.roleId" :label="role.roleId">{{ role.roleName }}</el-checkbox>
           </el-checkbox-group>
         </el-form-item>-->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data() {
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          departmentName: '',
          roleIdList: [],
          status: 1
        },
        dataRule: {}
      }
    },
    methods: {
      init (id, departmentName) {
        this.dataForm.id = id || 0
        this.dataForm.departmentName = departmentName
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/sys/department/${!this.dataForm.id ? 'save' : 'update'}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.dataForm.id || undefined,
            'departmentName': this.dataForm.departmentName
          })
        }).then(({data}) => {
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
      }
    }
  }
</script>
