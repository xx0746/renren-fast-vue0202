<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="500px"
  >
    <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="70px">
      <el-form-item label="系数月份" prop="code">
        <el-date-picker v-model="dataForm.showMonth" type="month" value-format="yyyy-MM" :clearable="false"
                        placeholder="选择月份" style="width: 390px" :disabled="true">
        </el-date-picker>
      </el-form-item>

      <el-form-item label="部门名称" prop="name">
        <el-select v-model="dataForm.departmentValue"
                   clearable placeholder="请选择部门"
                   style="width: 390px"
                   :clearable="false"
                   :disabled="disabledFlag"
        >
          <el-option
            v-for="item in departOptions"
            :key="item.id"
            :label="item.departmentName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="工资总额" prop="ut">
        <el-input v-model="dataForm.salary" placeholder="请填写数字" style="width: 390px"></el-input>
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
          id: 0,
          showMonth: '',
          departmentValue: '',
          salary: ''
        },
        departOptions: [],
        disabledFlag: false
      }
    },
    methods: {
      init (id, departmentId, salary, showMonth, departOptions) {
        this.dataForm.showMonth = showMonth
        this.departOptions = departOptions
        this.dataForm.id = id
        this.dataForm.departmentValue = departmentId
        this.dataForm.salary = salary
        if (id === 0) {
          this.disabledFlag = false
        } else {
          this.disabledFlag = true
        }
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        // let numReg = /^[0-9]*$/
        // let numRe = new RegExp(numReg)
        let salaryNum = Number(this.dataForm.salary)
        if (Number.isNaN(salaryNum)) {
          this.$message.error('请输入数字')
          return
        } else {
          if (salaryNum <= 0) {
            this.$message.error('请输入大于0的数字')
            return
          }
        }
        if (this.dataForm.departmentValue === '') {
          this.$message.error('请选择部门')
          return
        }
        let str = '修改'
        let url = 'updateSalary'
        let data = {}
        if (this.dataForm.id === 0) {
          str = '新增'
          url = 'addSalary'
          let departmentName = ''
          this.departOptions.some((item, i) => {
            if (item.id === this.dataForm.departmentValue) {
              departmentName = item.name
              return true
            }
          })
          data = {
            'year': this.dataForm.showMonth.substring(0, 4),
            'month': this.dataForm.showMonth.substring(5, 7),
            'departmentId': this.dataForm.departmentValue,
            'departmentName': departmentName,
            'salary': this.dataForm.salary
          }
        } else {
          data = {
            'id': this.dataForm.id,
            'salary': this.dataForm.salary
          }
        }
        this.$confirm('确定' + str + '吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl(`/departmentSalary/` + url),
            method: 'post',
            data: this.$http.adornData(data)
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
        }).catch(() => {
        })
      }
    }
  }
</script>
