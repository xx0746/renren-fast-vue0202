<template>
  <el-dialog
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="480px"
    :before-close="dataFormClose"
  >
    <el-form :model="dataForm"  ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="70px">
      <el-row style="margin-bottom: 0px;padding-bottom: 0px">
        <span>{{dataForm.dayvalue}}</span>
      </el-row>
      <el-row style="margin-top: 10px">
        <span>工作类型</span>
        <span style="margin-left: 176px">添加工时</span>
        <!--<el-col :span="8" ><el-form-item  label="工作类型" ></el-form-item> </el-col>
        <el-col :span="8" ><el-form-item style="margin-left: 70px" label="添加工时" ></el-form-item></el-col>-->
      </el-row>
      <el-row style="margin-top: 10px">
        <span>
          <el-cascader
            style="width: 200px"
            ref="cascader"
            :props="cascaderProp"
            v-model="dataForm.work"
            :options="optionWorks"
            clearable="true"
            @change="handleChange"></el-cascader>
        </span>
        <span style="margin-left: 30px">
          <el-select v-model="dataForm.workTime" filterable placeholder="请选择" style="width: 200px">
            <el-option
              v-for="item in optionTimes"
              :key="item.label"
              :label="item.value"
              :value="item.label">
            </el-option>
          </el-select>
        </span>
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
        nodesLable: [],
        nodesValue: '',
        cascaderProp: {value: 'id', label: 'workType', children: 'children'},
        optionWorks: [],
        optionTimes: [{
          value: '0',
          label: '0'
        }, {
          value: '0.5',
          label: '0.5'
        }, {
          value: '1',
          label: '1'
        }, {
          value: '1.5',
          label: '1.5'
        }, {
          value: '2',
          label: '2'
        }, {
          value: '2.5',
          label: '2.5'
        }, {
          value: '3',
          label: '3'
        }, {
          value: '3.5',
          label: '3.5'
        }, {
          value: '4',
          label: '4'
        }, {
          value: '4.5',
          label: '4.5'
        }, {
          value: '5',
          label: '5'
        }, {
          value: '5.5',
          label: '5.5'
        }, {
          value: '6',
          label: '6'
        }, {
          value: '6.5',
          label: '6.5'
        }, {
          value: '7',
          label: '7'
        }],
        dataForm: {
          projectId: '',
          dayvalue: '',
          work: [],
          workTime: 0,
          loading: false
        }
      }
    },
    methods: {
      init (projectId, dayvalue, optionWorks) {
        this.dataForm.projectId = projectId
        this.dataForm.dayvalue = dayvalue
        this.optionWorks = optionWorks
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        if (this.dataForm.workTime === '' || this.dataForm.workTime === undefined) {
          this.$message.error('工时必填')
          return
        }
        if (this.dataForm.work === [] || this.dataForm.work.length === 0) {
          this.$message.error('请选择工作类型')
          return
        }
        this.visible = false
        let workTime1 = this.dataForm.workTime
        this.dataForm.work = []
        this.dataForm.workTime = 0
        this.$emit('refreshDataList', this.nodesLable, this.nodesValue, workTime1)
      },
      dataFormClose () {
        // eslint-disable-next-line no-unused-expressions
        this.visible = false
        this.dataForm.work = []
        this.dataForm.workTime = 0
      },
      handleChange (val) {
        this.nodesLable = this.$refs['cascader'].currentLabels
        this.nodesValue = val
      }
    }
  }
</script>
