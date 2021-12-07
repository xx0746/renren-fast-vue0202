<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm"  ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="70px">
      <el-form-item label="项目编码" prop="code">
        <el-input v-model="dataForm.code" placeholder="项目编码" disabled></el-input>
      </el-form-item>

      <el-form-item label="项目名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="项目名称"  disabled></el-input>
      </el-form-item>
      <el-form-item label="项目工时" prop="ut">
        <el-input v-model="dataForm.ut" placeholder="项目工时"></el-input>
      </el-form-item>

    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: '',
          code: '',
          name: '',
          ut: 0
        }
      }
    },
    methods: {
      init (id, code, name, ut) {
        this.dataForm.id = id
        this.dataForm.code = code
        this.dataForm.name = name
        this.dataForm.ut = ut
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        let utNum = Number(this.dataForm.ut)
        if (Number.isNaN(utNum)) {
          this.$message.error('请输入数字')
          return
        } else {
          if (utNum <= 0) {
            this.$message.error('请输入大于0的数字')
            return
          }
        }
        this.$confirm(`确定修改吗?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl(`/project/updateProject`),
            method: 'post',
            data: this.$http.adornData({
              'id': this.dataForm.id,
              'ut': this.dataForm.ut
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
        }).catch(() => {})
      }
    }
  }
</script>
