<template>
  <div>
    <j-modal
      :visible.sync="modal.visible"
      :width="1200"
      :title="modal.title"
      :fullscreen.sync="modal.fullscreen"
      :switchFullscreen="modal.switchFullscreen"
      :ok-close="false"
      @ok="handleSubmit"
      @cancel="confirmCancel"
    >
      <a-form @submit="handleSubmit" :form="form" :label-col="labelCol" :wrapper-col="wrapperCol">
        <a-collapse default-active-key="1" :bordered="false" v-model="stepActiveKey" @change="openStepActive">
          <a-collapse-panel v-for="(item, key) in steps" :key="`${item.key}`" :style="customStyle">
            <div slot="header">
              <div style="display: flex;align-items:center">
                <div>
                  <a-avatar style="color: #FFFFFF; backgroundColor: #17C1C2">
                    {{ item.key }}
                  </a-avatar>
                </div>
                <div style="margin-left: 15px;color: #333333">{{ item.title }}</div>
              </div>
            </div>

            <div v-if="item.key === 1" class="steps-content">
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item
                    label="任务名">
                    <a-input placeholder="请输入任务名称"
                             v-decorator="[ 'name', {rules: [{ required: true, message: '请输入任务名称', whitespace: true}]} ]"/>
                  </a-form-item>
                </a-col>
                <a-col :span="12">
                  <a-form-item
                    label="描述">
                    <a-input placeholder="请输入描述"
                             v-decorator="[ 'desc', {rules: [{ required: false, message: '请输入描述', whitespace: true}]} ]"/>
                  </a-form-item>
                </a-col>
              </a-row>
            </div>


            <div v-if="item.key === 2" class="steps-content">
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item
                    label="数据源">
                    <a-select
                      placeholder="请选择数据源"
                      @change="updateOriginal"
                      v-decorator="[
              'originalId',
              {rules: [{ required: true, message: '请选择数据源'}]}
            ]">
                      <a-select-option v-for="(item,index) in originalList" :key="index" :value="item.id">
                        {{ item.dataSourceName }}
                      </a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="12">
                  <a-form-item
                    label="数据源表或视图">
                    <a-select
                      show-search
                      :filter-option="filterOption"
                      placeholder="请选择数据源表"
                      @change="confirmOriginalTable"
                      v-decorator="[
              'originalTable',
              {rules: [{ required: true, message: '请选择数据源表'}]}
            ]">
                      <a-select-option v-for="(item,index) in originalTableList" :key="index" :value="item">{{
                          item
                        }}
                      </a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
              </a-row>
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item
                    label="全量/增量" :wrapper-col="{ span: 8 }">
                    <div style="display:flex;align-items:center;">
                      <div>
                        <a-radio-group @change="switchJobType"
                                       v-decorator="['collectionType',{rules: [{ required: true, message: '请选择全量或者增量'}],'initialValue':0}]">
                          <a-radio-button :value="0">全量</a-radio-button>
                          <a-radio-button :value="1">增量</a-radio-button>
                        </a-radio-group>
                      </div>
                    </div>

                  </a-form-item>
                </a-col>
                <a-col :span="12">
                </a-col>
              </a-row>
              <div v-if="incColumnShow" style="display: flex;justify-content: center;padding-bottom: 30px;">
                <div style="display: flex;justify-content: start;width: 1001px;">
                  <div style="min-width:251px;max-width: 1001px;">
                    <div style="background: #cccccc;color: #FFFFFF;text-align: left;padding-left: 10px;">字段信息</div>
                    <div style="display: flex;text-align: left;border-left: 1px solid #cccccc;flex-wrap:wrap">
                      <div v-for="(item,index) in originalTableColumns" :key="index" v-if="item.incColumn===1"
                           v-model="incColumn" style="width: 250px;" @click="confirmIncColumn(item.columnName)">
                        <div class="incColumn">
                          <div style="display: flex">
                            <div style="width: 220px;">{{ item.columnName }}&nbsp;<span
                              style="color: #cccccf">({{ item.columnType }})</span></div>
                            <div>
                              <a-icon v-if="incColumn===item.columnName" style="color: #00DB00" type="check-circle"/>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <a-row type="flex" :gutter="1" v-if="collectionType===1">
                <a-col :span="12">
                  <!--                <a-form-item-->
                  <!--                  label="增量类型">-->
                  <!--                  <a-radio-group @change="switchIncrementType" v-decorator="['incrementType',{rules: [{ required: true, message: '请选择全量或者增量'}],'initialValue':0}]">-->
                  <!--                    <a-radio :value="0">自增ID</a-radio>-->
                  <!--                    <a-radio :value="1">自增时间戳</a-radio>-->
                  <!--                  </a-radio-group>-->
                  <!--                </a-form-item>-->

                  <a-form-item
                    label="全量/增量" :wrapper-col="{ span: 8 }">
                    <div style="display:flex;align-items:center;">
                      <div>
                        <a-radio-group @change="switchIncrementType"
                                       v-decorator="['incrementType',{rules: [{ required: true, message: '请选择全量或者增量'}],'initialValue':0}]">
                          <a-radio-button :value="0">自增ID</a-radio-button>
                          <a-radio-button :value="1">自增时间戳</a-radio-button>
                        </a-radio-group>
                      </div>
                    </div>
                  </a-form-item>
                </a-col>
              </a-row>
              <a-row type="flex" :gutter="1" v-if="collectionType===1">
                <a-col :span="12">
                  <a-form-item
                    label="自增ID" :wrapper-col="{ span: 8 }" v-if="incrementType===0">
                    <div style="display:flex;align-items:center;">
                      <div>
                        <a-input v-decorator="['incStartId',{'initialValue':0}]"></a-input>
                      </div>
                    </div>
                  </a-form-item>

                  <a-form-item
                    label="起始时间" :wrapper-col="{ span: 8 }" v-if="incrementType===1">
                    <div style="display:flex;align-items:center;">
                      <div>
                        <a-date-picker :show-time="{ format: 'HH:mm' }" format="YYYY-MM-DD HH:mm" valueFormat="YYYY-MM-DD HH:mm"
                                       v-decorator="['incStartTime',{'initialValue':0}]">
                        </a-date-picker>
                      </div>
                    </div>
                  </a-form-item>
                </a-col>
              </a-row>


            </div>


            <div v-if="item.key === 3" class="steps-content">
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item
                    label="目标端">
                    <a-select
                      placeholder="请选择目标端"
                      @change="updateTarget"
                      v-decorator="[
              'targetId',
              {rules: [{ required: true, message: '请选择目标端'}]}
            ]">
                      <a-select-option v-for="(item,index) in targetList" :key="index" :value="item.id">
                        {{ item.dataSourceName }}
                      </a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="12">
                  <a-form-item
                    label="目标表">
                    <a-select
                      placeholder="请选择目标表"
                      @change="confirmTargetTable"
                      v-decorator="[
              'targetTable',
              {rules: [{ required: true, message: '请选择目标表'}]}
            ]">
                      <div slot="dropdownRender" slot-scope="menu">
                        <v-nodes :vnodes="menu"/>
                        <a-divider style="margin: 4px 0;"/>
                        <div
                          style="padding: 4px 8px; cursor: pointer;"
                          @click="createTable"
                        >
                          <a-icon type="plus"/>
                          创建表
                        </div>
                      </div>
                      <a-select-option v-for="(item,index) in targetTableList" :key="index" :value="item">{{
                          item
                        }}
                      </a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
              </a-row>
            </div>


            <div v-if="item.key === 4" class="steps-content">
              <div style="display: flex;justify-content: center">
                <div style="width: 1200px;">
                  <div v-if="originalTableColumns.length > 0"
                       style="display: flex;align-items:center;padding-bottom: 30px;">
                    <div
                      style="margin-right: 50px;border: 1px solid #cccccc;border-radius: 5px;padding: 10px;height: 500px;width: 300px;overflow-y: auto">
                      <div style="font-weight: bold">数据源表</div>
                      <div v-for="(item,index) in originalTableColumns" :key="index"
                           v-bind:class="{checked:index==originalColumnIndex}" @click="clickOriginalColumn(index)">
                        <div v-if="item.show !== 0" style="display: flex;padding: 2px;border-radius: 4px;">
                          <div style="width:150px;margin-right: 50px;text-align: left;">{{ item.columnName }}</div>
                          <div style="width:80px;text-align: left">{{ item.columnType }}</div>
                        </div>
                      </div>
                    </div>
                    <div v-if="targetTableColumns.length > 0">
                      <div
                        style="margin-right: 50px;border: 1px solid #cccccc;border-radius: 5px;padding: 10px;height: 500px;width: 300px;overflow-y: auto">
                        <div style="font-weight: bold">目标端表</div>
                        <div v-for="(item,index) in targetTableColumns" :key="index"
                             v-bind:class="{checked:index==targetColumnIndex}" @click="clickTargetColumn(index)"
                             @dblclick="confirmMapping">
                          <div v-if="item.show !== 0" style="display: flex;">
                            <div style="width:150px;margin-right: 50px;text-align: left;">{{ item.columnName }}</div>
                            <div style="width:80px;text-align: left">{{ item.columnType }}</div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div v-if="originalTableColumns.length>0 && targetTableColumns.length > 0">
                      <a-button type="primary" style="margin-right: .8rem" @click="autoMatch">自动匹配</a-button>
                    </div>
                    <div v-if="mappingColumns.length > 0">
                      <div
                        style="margin-left: 35px;border: 1px solid #cccccc;border-radius: 5px;padding: 10px;height: 500px;overflow-y: auto">
                        <div style="font-weight: bold">映射</div>
                        <div v-for="(item,index) in mappingColumns" :key="index" style="display: flex"
                             v-bind:class="{checked:index==mappingColumnIndex}" @click="clickMappingColumn(index)"
                             @dblclick="cancelMapping(index)">
                          <div style="width:150px;text-align: right;padding: 3px;height: 15px;line-height: 15px;">
                            {{ item.originalColumn }}
                          </div>
                          <div>-></div>
                          <div style="width:150px;text-align: left;padding: 3px;height: 15px;line-height: 15px;">
                            {{ item.targetColumn }}
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>


            <div v-if="item.key === 5" class="steps-content">
              <a-row :gutter="16">
                <a-col :span="12">
                  <a-form-item
                    label="任务周期">
                    <j-cron ref="innerVueCron" v-decorator="['taskCorn', { initialValue: '0 0 0 * * ? *' }]"></j-cron>
                  </a-form-item>
                </a-col>
                <a-col :span="12">
                  <!--                <a-form-item-->
                  <!--                  label="描述">-->
                  <!--                  <a-input placeholder="请输入描述"-->
                  <!--                           v-decorator="[ 'desc', {rules: [{ required: false, message: '请输入描述', whitespace: true}]} ]"/>-->
                  <!--                </a-form-item>-->
                </a-col>
              </a-row>
            </div>

          </a-collapse-panel>
        </a-collapse>
      </a-form>
      <create-table ref="createTable" @addTable="addTable"></create-table>
    </j-modal>
  </div>

</template>

<script>
import { getDataSourceList, getDataSourceTables, getTableColumns, saveOfflineTask } from '@/api/api'
import { isNull } from 'xe-utils/methods'
import JCron from '@/components/jeecg/JCron.vue'
import CreateTable from './CreateTable'

export default {
  name: 'AddOfflineTask',
  components: {
    CreateTable,
    JCron,
    VNodes: {
      functional: true,
      render: (h, ctx) => ctx.props.vnodes,
    }
  },
  data() {
    return {
      modal: {
        title: '新增离线任务',
        visible: false,
        fullscreen: true,
        switchFullscreen: true,
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 12 },
      },
      incColumnShow: false,
      confirmLoading: false,
      stepActiveKey: [1],
      form: this.$form.createForm(this),
      current: 0,
      customStyle: 'border: 0;',
      incColumn: '',
      originalList: [],
      targetList: [],
      originalTableList: [],
      targetTableList: [],
      originalTableColumns: [],
      targetTableColumns: [],
      mappingColumns: [],
      originalColumnIndex: 0,
      targetColumnIndex: 0,
      mappingColumnIndex: 0,
      collectionType: 0,
      incrementType: 0,
      task: {},
      steps: [
        {
          key: 1,
          title: '填写基本信息',
          content: 'First-content',
        },
        {
          key: 2,
          title: '选择数据源',
          content: 'Second-content',
        },
        {
          key: 3,
          title: '选择目标端',
          content: 'Last-content',
        },
        {
          key: 4,
          title: '字段映射',
          content: 'Second-content',
        },
        {
          key: 5,
          title: '配置任务调度',
          content: 'Last-content',
        }
      ],
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    show() {
      this.modal.visible = true
    },
    init() {
      const that = this
      getDataSourceList({ 'purpose': 1 }).then((res) => {
        if (res.success) {
          that.originalList = res.result
        }
      })
      getDataSourceList({ 'purpose': 2 }).then((res) => {
        if (res.success) {
          that.targetList = res.result.filter((e)=>{return e.dbType === 'HIVE'})
        }
      })
    },
    updateOriginal(v, e) {
      this.task.originalId = v
      const that = this
      getDataSourceTables(v + '-0').then((res) => {
        if (res.success) {
          that.originalTableList = res.result
        }
      })
    },
    updateTarget(v, e) {
      this.task.targetId = v
      const that = this
      getDataSourceTables(v + '-1').then((res) => {
        if (res.success) {
          that.targetTableList = res.result
        }
      })
    },
    confirmOriginalTable(e) {
      console.log(this.task.originalId)
      this.task.originalTable = e
      const that = this
      console.log(that.stepActiveKey)
      const params = {
        'id': this.task.originalId,
        'tableName': e
      }
      getTableColumns(params).then((res) => {
        if (res.success) {
          that.originalTableColumns = res.result
        }
      })
      that.stepActiveKey.push(4)
    },
    confirmTargetTable(e) {
      const that = this
      const params = {
        'id': this.task.targetId,
        'tableName': e
      }
      getTableColumns(params).then((res) => {
        if (res.success) {
          that.targetTableColumns = res.result
        }
      })
    },
    clickOriginalColumn(index) {
      this.originalColumnIndex = index
    },
    clickTargetColumn(index) {
      this.targetColumnIndex = index
    },
    clickMappingColumn(index) {
      this.mappingColumnIndex = index
    },
    switchJobType(e) {
      this.collectionType = e.target.value
      if (e.target.value === 1) {
        this.incColumnShow = true
      } else {
        this.incColumnShow = false
      }
    },
    switchIncrementType(e) {
      this.incrementType = e.target.value
    },
    confirmIncColumn(e) {
      this.incColumn = e
    },
    confirmMapping() {
      // console.log(this.originalColumnIndex)
      if (this.originalColumnIndex === '') {
        this.$message.warning('请选择字段')
        return
      }
      if (this.targetColumnIndex === '') {
        this.$message.warning('请选择字段')
        return
      }
      // console.log(this.originalTableColumns[this.originalColumnIndex])
      // console.log(this.targetTableColumns[this.targetColumnIndex])
      const mapping = {
        originalColumn: this.originalTableColumns[this.originalColumnIndex].columnName,
        targetColumn: this.targetTableColumns[this.targetColumnIndex].columnName,
        originalColumnIndex: this.originalColumnIndex,
        targetColumnIndex: this.targetColumnIndex
      }
      this.originalTableColumns[this.originalColumnIndex].show = 0
      this.targetTableColumns[this.targetColumnIndex].show = 0
      this.originalColumnIndex = ''
      this.targetColumnIndex = ''
      this.mappingColumns.push(mapping)
    },
    cancelMapping(index) {
      const mapping = this.mappingColumns[index]
      this.originalTableColumns[mapping.originalColumnIndex].show = 1
      this.targetTableColumns[mapping.targetColumnIndex].show = 1
      this.mappingColumns.splice(index, 1)
    },
    openStepActive(e) {
      this.stepActiveKey = e
      // this.stepActiveKey.push(e[0])
    },
    confirmCancel() {
      this.$emit('close')
    },
    handleSubmit(e) {
      // console.log('submit')
      e.preventDefault()
      this.confirmLoading = true
      const that = this
      this.form.validateFields((err, values) => {
        console.log(!err)
        if (!err) {
          // eslint-disable-next-line no-console
          values.mappingColumns = this.mappingColumns
          values.incColumn = this.incColumn
          console.log('Received values of form: ', values)
          this.content = JSON.stringify(values)
          saveOfflineTask(values).then((res) => {
            if (res.success) {
              that.confirmLoading = false
              that.$message.success('保存成功')
              that.modal.visible = false
              this.$emit('close')
            } else {
              that.$message.error(res.message)
            }
          })
        }
      })
    },
    autoMatch() {
      const that = this
      that.mappingColumns = []
      let count = 0
      that.originalTableColumns.forEach(function (original,originalIndex) {
        that.targetTableColumns.forEach(function (target,targetIndex) {
          if(original.columnName == target.columnName){
            const mapping = {
              originalColumn: original.columnName,
              targetColumn: target.columnName,
              originalColumnIndex: originalIndex,
              targetColumnIndex: targetIndex
            }
            original.show = 0
            target.show = 0
            that.mappingColumns.push(mapping)
            count++
          }
        })
      })
      if(count===0){
        this.$message.warning('无匹配字段请手动匹配')
      }
    },
    createTable(){
      if(!this.task.originalId){
        this.$message.warning('请选择数据源');
        return
      }
      if(!this.task.originalTable){
        this.$message.warning('请选择源数据表');
        return
      }
      this.$refs.createTable.init(this.task)
    },
    addTable(tableName){
      this.targetTableList.push(tableName)
      console.log(this.targetTableList)
      this.form.setFieldsValue({targetTable:`${tableName}`})
      this.confirmTargetTable(tableName)
    },
    filterOption(input, option) {
      return (
        option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
      );
    },
  }
}
</script>

<style scoped>
.steps-content {
  margin-top: 16px;
  border: 1px dashed #e9e9e9;
  border-radius: 6px;
  background-color: #FFFFFF;
  min-height: 100px;
  text-align: center;
  padding: 30px 30px 0px 30px;
}

.steps-action {
  margin-top: 24px;
}

.checked {
  background: #00A0E9;
  color: #FFFFFF;
}

.incColumn {
  height: 25px;
  line-height: 25px;
  border-right: 1px solid #cccccc;
  border-bottom: 1px solid #cccccc;
  padding-left: 4px;
}

.incColumnChecked {
  height: 25px;
  line-height: 25px;
  border-left: 1px solid #cccccc;
  border-bottom: 1px solid #cccccc;
  padding-left: 4px;
  background: #8cc8ff;
  color: #FFFFFF;
}
</style>