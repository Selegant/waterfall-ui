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
        <a-button v-if="!isDetail" type="primary" :loading="syncLoading" @click="handleDbSync" >确定</a-button>
      </template>
      <div class="table-page-search-wrapper warp" :style="{maxHeight:fullscreen?'779px':'588px'}">
        <a-form-model ref="ruleForm" :model="formData" :rules="rules" :label-col="labelCol" :wrapper-col="wrapperCol">
            <a-form-model-item v-if="isImport" label="数据源" prop="source">
                <a-select placeholder="请选择数据源" @change="updateOriginal" v-model="formData.source">
                  <a-select-option v-for="item in originalList" :key="item.id" :value="item.id">
                    {{ item.dataSourceName }}
                  </a-select-option>
                </a-select>
            </a-form-model-item>
            <a-row>
              <a-col :span="16">
                <a-form-model-item v-if="isImport" label="表或视图" prop="tableName" :label-col="{span:3}" :wrapper-col="{span:21}">
                      <a-select placeholder="请选择表或视图" v-model="formData.tableName">
                        <a-select-option v-for="item in originalTableList" :key="item" :value="item">{{
                            item
                          }}
                        </a-select-option>
                      </a-select>
                </a-form-model-item>
              </a-col>
              <a-col :span="2" :offset="1">
                <a-button v-if='isImport' type="primary"  @click.prevent="importModel" >导入</a-button>
              </a-col>
            </a-row>
            <a-form-model-item label="模型名称" prop="modelName">
                <span v-if="isDetail">{{formData.modelName}}</span>
                <a-input v-else v-model="formData.modelName" placeholder="请输入模型名称"/>
            </a-form-model-item>
            <a-form-model-item label="模型描述" prop="remark">
                <span v-if="isDetail">{{formData.remark}}</span>
                <a-textarea v-else v-model="formData.remark" placeholder="请输入模型描述信息" allow-clear :auto-size="{ minRows: 3, maxRows: 5 }" />
            </a-form-model-item>
        </a-form-model>
        <a-tabs defaultActiveKey="1" type="card">
          <a-tab-pane key="1" tab="字段信息">
            <!-- 数据字段 -->
            <div class="table-operator">
              <p class="title">数据字段：</p>
              <div v-if="!isDetail">
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
              :row-selection="isDetail?null:{selectedRowKeys: dataSelectedRowKeys, onChange: onDataSelectChange}"
            >
              <div slot="fieldName" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['fieldName']"
                />
              </div>
              <div slot="fieldTypeName" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['fieldTypeName']"
                />
              </div>
              <div slot="remark" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['remark']"
                />
              </div>
              <div slot="primarykeyFlag" slot-scope="text, record">
                <span v-if="isDetail">{{text?'是':'否'}}</span>
                <a-checkbox v-else :checked="record['primarykeyFlag']?true:false" @change="checkboxChange(record,'primarykeyFlag')"  style="margin: -5px 0" ></a-checkbox>
              </div>
              <div slot="emptyFlag" slot-scope="text, record">
                <span v-if="isDetail">{{text?'是':'否'}}</span>
                <a-checkbox v-else :checked="record['emptyFlag']?true:false" @change="checkboxChange(record,'emptyFlag')"  style="margin: -5px 0" ></a-checkbox>
              </div>
              <div slot="length" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
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
              <div v-if="!isDetail">
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
              :row-selection="isDetail?null:{selectedRowKeys: partitionsSelectedRowKeys, onChange: onPartitionsSelectChange}"
            >
              <div slot="partitionName" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['partitionName']"
                />
              </div>
              <div slot="partitionTypeId" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
                  type="number"
                  style="margin: -5px 0"
                  :value="text"
                  v-model="record['partitionTypeId']"
                />
              </div>
              <div slot="remark" slot-scope="text, record">
                <span v-if="isDetail">{{text}}</span>
                <a-input
                  v-else
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
import { getDataModuleDetail, getDataSourceList, getDataSourceTables, getModalByTable } from '@/api/api'
import { postAction, putAction, getAction } from '@/api/manage'
export default {
  name: 'AddModelModal',
  data() {
    return {
      originalTableList:[],
      originalList:[],
      isImport:false,
      isDetail:false,
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
        // {
        //   title: '操作',
        //   align: 'center',
        //   dataIndex: 'action',
        //   width: '8%',
        //   scopedSlots: { customRender: 'action' }
        // }
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
        // {
        //   title: '操作',
        //   align: 'center',
        //   dataIndex: 'action',
        //   width: '8%',
        //   scopedSlots: { customRender: 'action' }
        // }
      ],
      url: {
        edit: '/modelManagement/data-module',
        add: '/modelManagement/data-module'
      },
      rules: {
        modelName: [
          { required: true, message: '请输入模型名称', trigger: 'change' }
        ],
        source: [
          { required: true, message: '请选择数据源', trigger: 'change' }
        ],
        tableName: [
          { required: true, message: '请选择表或视图', trigger: 'change' }
        ]
      },
      formData: {
        source:null,
        tableName:'',
        modelName:'',
        remark:'',
        modelFields:[],
        modelPartitions:[],
        folderId:null,
        id:null
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
  mounted() {
  },
  methods: {
    importModel(){
      const valiSome = ['source','tableName']
      let flag = true
      valiSome.forEach(item=>{
        this.$refs['ruleForm'].validateField(item,valid=>{
          if(valid.length){
            return flag = false
          }
        })
      })
      if(flag){
        let {folderId,source,tableName}=this.formData
        let params={
          folderId,
          tableName,
          source
        }
        getModalByTable(params).then(res=>{
          if(res.success){
            this.formData = {
              ...this.formData,
              ...res.result
            }
          }
        })
      }
    },
    updateOriginal(v) {
      this.formData.source = v
      const that = this
      getDataSourceTables(v + '-0').then((res) => {
        if (res.success) {
          that.originalTableList = res.result
        }
      })
    },
    getOriginalList(){
      getDataSourceList({purpose:1}).then(res=>{
        this.originalList=res.result || []
      })
    },
    handleCancleDbSync() { // 关闭
      this.formData = this.$options.data().formData
      this.isDetail = false
      this.isImport = false
      this.editModalVisible = false
    },
    edit() {
      this.editModalVisible = true
      this.getDataDetail()
    },
    add() {
      console.log(this.isImport,'import');
      this.editModalVisible = true
      if(this.isImport){
        this.getOriginalList()
      }
    },
    getDataDetail(){
      console.log(this.formData.id,this.isDetail,'id');
      getDataModuleDetail({id:this.formData.id}).then((res) => {
        if(res.result){
          Object.keys(this.formData).forEach((key)=>{
            this.formData[key] = res.result[key]
          })
        }
      })
    },
    async handleDbSync() { // 弹窗点击确定
        const valid = await this.$refs['ruleForm'].validate()
        if (valid) {
            this.updateData()
        }
    },
    updateData() { // 保存提交数据
      this.syncLoading = true
      const fun = (res) => {
          if (res.success) {
            this.$message.success(res.message)
            this.handleCancleDbSync()
            this.$emit('modalFormOk')
          } else {
            this.$message.warning(res.message)
          }
      }
      if (this.formData.id) {
        //编辑数据模型
        putAction(this.url.edit, this.formData).then(
          fun
        ).finally(() => {
          this.syncLoading = false
        })
      } else {
        //新增数据模型
        postAction(this.url.add, this.formData).then(
          fun
        ).finally(() => {
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
.table-page-search-wrapper{
  overflow:auto;
}
// /deep/ .ant-modal-body{
//   max-height:636px;
//   overflow:auto;
// }
</style>