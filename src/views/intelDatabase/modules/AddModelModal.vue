<template>
<div>
    <j-modal
      width="80%"
      :title="title"
      :visible.sync="editModalVisible"
      :fullscreen.sync="fullscreen"
      :switch-fullscreen="switchFullscreen"
      @cancel="handleCancleDbSync"
      :destroyOnClose="true"
    >
      <template slot="footer">
        <a-button @click="handleCancleDbSync">
          关闭
        </a-button>
        <a-button type="primary" :loading="syncLoading" @click="handleDbSync" >确定</a-button>
      </template>
      <div class="table-page-search-wrapper warp">
        <a-form-model ref="ruleForm" :model="formData" :rules="rules" :label-col="labelCol" :wrapper-col="wrapperCol">
            <a-form-model-item label="模型名称" prop="modelName">
                <a-input v-model="formData.modelName" placeholder="请输入模型名称"/>
            </a-form-model-item>
            <a-form-model-item label="模型描述" prop="remark">
                <a-textarea v-model="formData.remark" placeholder="请输入模型描述信息" allow-clear :auto-size="{ minRows: 3, maxRows: 5 }" />
            </a-form-model-item>
        </a-form-model>
        <a-tabs defaultActiveKey="1" type="card">
          <a-tab-pane key="1" tab="字段信息">
            <!-- 数据字段 -->
            <div class="table-operator">
              <p class="title">数据字段：</p>
              <div>
                <a-button type="link" icon="plus" @click="handleAddModelFields">新增数据字段</a-button>
                <a-button type="link" v-if="dataSelectedRowKeys.length" icon="minus" @click="handleRemoveModelFields('dataSelectedRowKeys','modelFields')">删除</a-button>
              </div>
            </div>
            <a-table
              style="margin-bottom:20px;"
              size="middle"
              bordered
              :rowKey="(record, index) => index"
              :columns="columns"
              :data-source="formData.modelFields"
              :pagination="false"
              :row-selection="{selectedRowKeys: dataSelectedRowKeys, onChange: onDataSelectChange}"
            >
              <div slot="fieldName" slot-scope="text, record">
                <a-input
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['fieldName']"
                />
              </div>
              <div slot="fieldTypeName" slot-scope="text, record">
                <a-input
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['fieldTypeName']"
                />
              </div>
              <div slot="remark" slot-scope="text, record">
                <a-input
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['remark']"
                />
              </div>
              <div slot="primarykeyFlag" slot-scope="text, record">
                <a-checkbox :checked="record['primarykeyFlag']?true:false" @change="checkboxChange(record,'primarykeyFlag')"  style="margin: -5px 0" ></a-checkbox>
              </div>
              <div slot="emptyFlag" slot-scope="text, record">
                <a-checkbox :checked="record['emptyFlag']?true:false" @change="checkboxChange(record,'emptyFlag')"  style="margin: -5px 0" ></a-checkbox>
              </div>
              <div slot="length" slot-scope="text, record">
                <a-input
                  type="number"
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['length']"
                />
              </div>
            </a-table>
            <!-- 分区字段 -->
            <div class="table-operator">
              <p class="title">分区字段：</p>
              <div>
                <a-button type="link" icon="plus" @click="handleAddModelPartitions">新增分区字段</a-button>
                <a-button type="link" v-if="partitionsSelectedRowKeys.length" icon="minus" @click="handleRemoveModelFields('partitionsSelectedRowKeys','modelPartitions')">删除</a-button>
              </div>
            </div>
            <a-table
              size="middle"
              bordered
              :rowKey="(record, index) => index"
              :columns="partitionsColumns"
              :data-source="formData.modelPartitions"
              :pagination="false"
              :row-selection="{selectedRowKeys: partitionsSelectedRowKeys, onChange: onPartitionsSelectChange}"
            >
              <div slot="partitionName" slot-scope="text, record">
                <a-input
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['partitionName']"
                />
              </div>
              <div slot="partitionTypeId" slot-scope="text, record">
                <a-input
                type="number"
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['partitionTypeId']"
                />
              </div>
              <div slot="remark" slot-scope="text, record">
                <a-input
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['remark']"
                />
              </div>
            </a-table>
          </a-tab-pane>
        </a-tabs>
      </div>
    </j-modal>
    </div>
</template>
<script>
import { postAction, putAction, getAction } from '@/api/manage'
export default {
  name: 'AddModelModal',
  data() {
    return {
      labelCol: { span: 2 },
      wrapperCol: { span: 14 },
      editModalVisible: false,
      title: '编辑',
      syncLoading: false,
      fullscreen: true,
      switchFullscreen: true,
      dataSelectedRowKeys: [],
      partitionsSelectedRowKeys:[],
      partitionsColumns:[
          {
          title: '字段名称',
          align: 'center',
          width: '15%',
          dataIndex: 'partitionName',
          scopedSlots: { customRender: 'partitionName' }
        },
        {
          title: '字段类型',
          align: 'center',
          width: '12%',
          dataIndex: 'partitionTypeId',
          scopedSlots: { customRender: 'partitionTypeId' }
        },
        {
          title: '描述',
          align: 'center',
          dataIndex: 'remark',
          scopedSlots: { customRender: 'remark' }
        },
        {
          title: '关联数据元',
          align: 'center',
          dataIndex: 'metadataId',
          width: '12%',
        //   scopedSlots: { customRender: 'metadataId' }
        },
        {
          title: '操作',
          align: 'center',
          dataIndex: 'action',
          width: '8%',
          scopedSlots: { customRender: 'action' }
        }
      ],
      columns: [
        {
          title: '字段名称',
          align: 'center',
          width: '15%',
          dataIndex: 'fieldName',
          scopedSlots: { customRender: 'fieldName' }
        },
        {
          title: '字段类型',
          align: 'center',
          width: '12%',
          dataIndex: 'fieldTypeName',
          scopedSlots: { customRender: 'fieldTypeName' }
        },
        {
          title: '描述',
          align: 'center',
          dataIndex: 'remark',
          scopedSlots: { customRender: 'remark' }
        },
        {
          title: '关联数据元',
          align: 'center',
          dataIndex: 'metadataId',
          width: '12%',
        //   scopedSlots: { customRender: 'metadataId' }
        },
        {
          title: '是否为主键',
          align: 'center',
          dataIndex: 'primarykeyFlag',
          width: '12%',
          scopedSlots: { customRender: 'primarykeyFlag' }
        },
        {
          title: '是否为空',
          align: 'center',
          dataIndex: 'emptyFlag',
          width: '12%',
          scopedSlots: { customRender: 'emptyFlag' }
        },
        {
          title: '长度',
          align: 'center',
          dataIndex: 'length',
          width: '8%',
          scopedSlots: { customRender: 'length' }
        },
        {
          title: '操作',
          align: 'center',
          dataIndex: 'action',
          width: '8%',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/online/cgform/head/list',
        edit: '/diagram/diagramCombination/edit',
        add: '/modelManagement/data-module'
      },
      rules: {
        modelName: [
          { required: true, message: '请输入模型名称', trigger: 'change' }
        ]
      },
      formData: {
        modelName:'',
        remark:'',
        modelFields:[],
        modelPartitions:[],
        folderId:null
      },
      newModelFieldsData: {
        fieldName:'',
        fieldTypeName:'',
        remark:'',
        metadataId:null,
        primarykeyFlag:1,
        emptyFlag:0,
        length:null
      },
      newModelPartitionsData:{
        partitionName:'',
        partitionTypeId:null,
        remark:'',
        metadataId:null
      }
    }
  },
  methods: {
    handleCancleDbSync() { // 关闭
      this.formData = this.$options.data().formData
      this.editModalVisible = false
    },
    edit(record) {
      var that = this
      const record_ = JSON.parse(JSON.stringify(record))
      Object.keys(that.formData).forEach(function(key) {
        that.formData[key] = record_[key]
      })
      this.editModalVisible = true
    },
    add() {
      this.editModalVisible = true
    },
    async handleDbSync() { // 弹窗点击确定
        const valid = await this.$refs['ruleForm'].validate()
        if (valid) {
            this.updateData()
        }
    },
    updateData(params) { // 保存提交数据
      this.syncLoading = true
      if (this.formData.id) {
        //编辑数据模型
        putAction(this.url.edit, this.formData).then((res) => {
          if (res.success) {
            this.$message.success(res.message)
            this.editModalVisible = false
            this.$emit('modalFormOk')
          } else {
            this.$message.warning(res.message)
          }
        }).finally(() => {
          // this.loading = false
        })
      } else {
        //新增数据模型
        postAction(this.url.add, this.formData).then((res) => {
          if (res.success) {
            this.$message.success(res.message)
            this.editModalVisible = false
            this.$emit('modalFormOk')
          } else {
            this.$message.warning(res.message)
          }
        }).finally(() => {
          this.syncLoading = false
        })
      }
    },
    handleAddModelPartitions() { // 新增分区字段
      const newData = {...this.newModelPartitionsData}
      this.formData.modelPartitions.push(newData)
    },
    handleAddModelFields() { // 新增数据字段
      const newData = {...this.newModelFieldsData}
      this.formData.modelFields.push(newData)
    },

    handleRemoveModelFields(rowKeysName,modelName) { // 删除数据字段
      var that = this
      this[rowKeysName].sort(function(a, b) { return b - a })
      this[rowKeysName].forEach(function(index) {
        that.formData[modelName].splice(index, 1)
      })
      this[rowKeysName] = []
    },
    onPartitionsSelectChange(partitionsSelectedRowKeys){
        this.partitionsSelectedRowKeys = partitionsSelectedRowKeys
    },
    onDataSelectChange(dataSelectedRowKeys) {
      this.dataSelectedRowKeys = dataSelectedRowKeys
    },
    checkboxChange(data, name) {
      if (data[name]) {
        data[name] = 0
      } else {
        data[name] = 1
      }
    }
  }
}
</script>
<style lang="less" scoped>
.warp .ant-select{
  width: 100%;
}
.warp .table-operator{
//   margin-bottom: 10px;
  display:flex;
  justify-content: space-between;
  line-height:35px;
  .title{
      margin-bottom:0;
  }
}
.ant-card-body .warp {
  padding: 10px;
}
.warp .table-operator .ant-btn{
  margin: 0px 8px 0 0 !important;
}
.ant-card-body .warp .table-operator{
  margin-bottom:10px !important;
}
/deep/ .ant-tabs.ant-tabs-card .ant-tabs-card-bar .ant-tabs-tab-active{
    border-top: 3px solid #1890FF;
}
</style>