<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible" width="500px">
    <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="70px">
      <el-row>
        <el-col :span="8">
          <el-form-item label="添加员工"></el-form-item>
        </el-col>
        <el-col :span="8">
          <el-form-item style="margin-left: 70px" label="添加工时"></el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="8">
          <el-select
            v-model="dataForm.person"
            clearable="true"
            filterable
            remote
            reserve-keyword
            onchange="getPersonId"
            placeholder="请输入员工"
            :remote-method="remoteMethod"
            :loading="loading">
            <el-option
              v-for="item in options"
              :key="item.userId"
              :label="item.userName"
              :value="item.userId">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="8" style="margin-left: 70px">
          <el-input v-model="dataForm.personut" placeholder="项目工时"></el-input>
        </el-col>
      </el-row>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="dataFormClose()">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        options: [],
        dataForm: {
          projectId: '',
          unuserut: 0,
          person: '',
          personut: '',
          loading: false
        }
      }
    },
    methods: {
      init (projectId, unuserut) {
        this.dataForm.projectId = projectId
        this.dataForm.unuserut = unuserut
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        let personUTNum = Number(this.dataForm.personut)
        if (Number.isNaN(personUTNum)) {
          this.$message.error('请输入数字')
          return
        } else {
          if (personUTNum <= 0) {
            this.$message.error('请输入大于0的数字')
            return
          }
        }
        if (personUTNum > this.dataForm.unuserut) {
          this.$message.info('分配工时大于剩余工时，无法新增')
          return
        }
        if (this.dataForm.person === '' || this.dataForm.person === null) {
          this.$message.info('请选择员工')
          return
        }
        this.$http({
          url: this.$http.adornUrl(`/userProject/addPerson`),
          method: 'post',
          params: this.$http.adornParams({
            'projectId': this.dataForm.projectId,
            'userId': this.dataForm.person,
            'ut': this.dataForm.personut
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.options = []
                this.dataForm.person = ''
                this.dataForm.personut = ''
                this.$emit('refreshDataList')
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      dataFormClose() {
        this.visible = false
        this.options = []
        this.dataForm.person = ''
        this.dataForm.personut = ''
      },
      remoteMethod (query) {
        if (query !== '') {
          this.loading = true
          this.$http({
            url: this.$http.adornUrl('/sys/user/userList'),
            method: 'post',
            params: this.$http.adornParams({
              'current': 1,
              'size': 50,
              'userName': query
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.options = data.dataList
            }
            this.loading = false
          })
        } else {
          this.options = []
        }
      }
    }
  }
</script>
