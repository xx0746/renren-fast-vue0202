<template >
  <el-dialog
    title="上传文件"
    :close-on-click-modal="false"
    @close="closeHandle"
    :visible.sync="visible">
    <el-form  :model="dataForm" >
      <el-form-item style="text-align: center;" >
        <span>导入月份:</span>
        <el-date-picker  v-model="dataForm.excelMonth" type="month" value-format="yyyy-MM"
                        placeholder="选择上传月份" style="width: 120px" :clearable="false">
        </el-date-picker>
      </el-form-item>
      <el-upload
        drag
        :action="url"
        :before-upload="beforeUploadHandle"
        :on-success="successHandle"
        multiple
        style="text-align: center;">
        <i class="el-icon-upload"></i>
        <div class="el-upload__text"><em>点击上传</em></div>
      </el-upload>
    </el-form>


  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          excelMonth: ''
        },
        url: '',
        num: 0,
        successNum: 0,
        fileList: []
      }
    },
    methods: {
      init (excelMonth) {
        let fullYear = new Date().getFullYear()
        let month = new Date().getMonth() + 1
        if (month < 10) {
          month = '0' + month
        }
        this.dataForm.excelMonth = fullYear + '-' + month
        this.url = this.$http.adornUrl(`/project/projectUpload?excelMonth=${this.dataForm.excelMonth}&token=${this.$cookie.get('token')}`)
        this.visible = true
      },
      // 上传之前
      beforeUploadHandle (file) {

      },
      // 上传成功
      successHandle (response, file, fileList) {
        if (response && response.code === 0) {
          this.$message.info('导入成功')
          this.visible = false
          this.$emit('refreshDataList')
        } else {
          this.$message.error(response.msg)
        }
      },
      // 弹窗关闭时
      closeHandle () {

      }
    }
  }
</script>
