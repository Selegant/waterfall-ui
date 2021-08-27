<template>
  <div style="width:100%;">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <!-- 搜索区域 -->
      <a-form-model layout="inline" :model="params" @keyup.enter.native="search">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-model-item style="margin: 0px;" label="关键字" :labelCol="{span: 5}" :wrapperCol="{span: 5, offset: 1}">
              <a-input placeholder="请输入关键字" v-model="params.modelName" ></a-input>
            </a-form-model-item>
          </a-col>
          <!--
          <a-col :md="11" :sm="12">
            <a-form-item label="创建时间" :labelCol="{span: 5}" :wrapperCol="{span: 18, offset: 1}">
              <j-date v-model="queryParam.createTime_begin" :showTime="true" date-format="YYYY-MM-DD HH:mm:ss" style="width:45%" placeholder="请选择开始时间" ></j-date>
              <span style="width: 10px;">~</span>
              <j-date v-model="queryParam.createTime_end" :showTime="true" date-format="YYYY-MM-DD HH:mm:ss" style="width:45%" placeholder="请选择结束时间"></j-date>
            </a-form-item>
          </a-col>
          -->
          <span style="" class="">
            <a-col :md="8" :sm="8">
              <a-button type="primary" icon="search" style="margin-left: 21px" @click="search">查询</a-button>
              <a-button icon="reload" style="margin-left: 8px" @click="reset">重置</a-button>
            </a-col>
          </span>

          <span style="display: flex;justify-content: flex-end" class="">
            <a-col>
              <a-button icon="vertical-align-bottom" style="margin-left: 21px" @click="importDataModule">导入数据模型</a-button>
              <a-button type="primary"  icon="plus" style="margin-left: 8px" @click="addDataModel">新建数据模型</a-button>
            </a-col>
          </span>
        </a-row>
      </a-form-model>
    </div>

    <div>
      <a-table
        bordered
        size="middle"
        :columns="columns"
        :data-source="data"
        :pagination="pagination"
        :loading="loading"
        @change="handleTableChange"
      >
      <span slot="modelStatus" slot-scope="text, record">
        <a-tag v-if="record.modelStatusCode===1" color="gray">未发布</a-tag>
        <a-tag v-if="record.modelStatusCode===2" color="blue">已发布</a-tag>
      </span>

        <span slot="action" slot-scope="text, record">
              <a @click="editDataModel(record.id)">编辑</a>
              <a-divider type="vertical"/>
              <a @click="handleEdit(record)">质量</a>
              <a-divider type="vertical"/>
              <a @click="publishDataModule(record)">{{record.modelStatusCode===1?'发布':record.modelStatusCode===2?'下架':''}}</a>
              <a-divider type="vertical"/>
              <a-dropdown>
                <a class="ant-dropdown-link">
                  更多 <a-icon type="down"/>
                </a>
                <a-menu slot="overlay">
                  <a-menu-item>
                    <a href="javascript:;" @click="detailClick(record.id)">详情</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a-popconfirm title="确定删除吗?" @confirm="() => deleteDataModule(record.id)">
                      <a>删除</a>
                    </a-popconfirm>
                  </a-menu-item>
                </a-menu>
              </a-dropdown>
            </span>
      </a-table>
    </div>
    <add-model-modal ref="modalForm" @modalFormOk="modalFormOk"></add-model-modal>
  </div>
</template>

<script>
import { getDataModuleList, publishDataModule, unpublishDataModule, deleteDataModule } from '../../../api/api'
import AddModelModal from './AddModelModal'
const queryData = params => {
  return axios.get('https://randomuser.me/api', { params: params })
}
const columns = [
  {
    title: '模型名',
    dataIndex: 'modelName',
    width: '300px'
  },
  {
    title: '描述',
    dataIndex: 'remark',
    width: '300px'
  },
  {
    title: '质量',
    dataIndex: 'exportTypeName',
    align: 'center',
    width: '80px'
  },
  {
    title: '发布状态',
    dataIndex: 'modelStatusCode',
    scopedSlots: {customRender: 'modelStatus'},
    align: 'center',
    width: '80px'
  },
  {
    title: '操作',
    scopedSlots: { customRender: 'action' },
    align: 'center',
    width: '180px'
  },
]

export default {
  name: 'ModelList',
  components: { AddModelModal },
  data() {
    return {
      data: [],
      pagination: {
        current: 1,
        pageSize: 10,
        pageSizeOptions: ['10', '20', '30'],
        showTotal: (total, range) => {
          return  " 共" + total + "条"
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      loading: false,
      columns,
      params: {
        modelName:'',
        folderId:0
      }
    }
  },
  mounted() {
    this.loadData()
  },
  methods: {
    deleteDataModule(id){
      deleteDataModule({id}).then(res=>{
        if(res.success){
          this.$message.success(res.message)
          this.loadData()
        }else{
          this.$message.error(res.message)
        }
      })
    },
    publishDataModule({modelStatusCode,id}){
      console.log(id,'发布id');
      const fu = res=>{
        if(res.success){
          this.$message.success(res.message)
          this.loadData()
        }else{
          this.$message.warning(res.message)
        }
      }
      if(modelStatusCode==1){
        publishDataModule({id}).then(
          fu
        )
      }
      if(modelStatusCode==2){
        unpublishDataModule({id}).then(
          fu
        )
      }
    },
    addDataModel(){
      this.$refs.modalForm.title = "新建-数据模型";
      this.$refs.modalForm.formData.folderId=Number(this.params.folderId)
      this.$refs.modalForm.add();
    },
    editDataModel(id){
      this.$refs.modalForm.title = "编辑-数据模型";
      this.$refs.modalForm.formData.folderId=Number(this.params.folderId)
      this.$refs.modalForm.formData.id=id
      this.$refs.modalForm.edit();
    },
    detailClick(id){
      this.$refs.modalForm.title = "详情-数据模型";
      this.$refs.modalForm.formData.id=id
      this.$refs.modalForm.isDetail=true
      this.$refs.modalForm.edit();
    },
    importDataModule(){
      this.$refs.modalForm.title = "导入数据模型";
      this.$refs.modalForm.formData.folderId=Number(this.params.folderId)
      this.$refs.modalForm.isImport=true
      this.$refs.modalForm.add();
    },
    modalFormOk(){
      // 新增、编辑成功刷新列表数据
      this.loadData()
    },
    handleTableChange(pagination, filters, sorter) {
      console.log(pagination,'pagination');
      this.pagination = {...pagination}
      this.loadData()
    },
    search(){
      this.pagination.current=1
      this.loadData()
    },
    reset(){
      this.params.modelName=''
      this.pagination.current=1
      this.loadData()
    },
    loadData(){
      this.loading = true
      this.params = {
        ...this.params,
        pageNo:this.pagination.current,
        pageSize:this.pagination.pageSize
      }
      console.log(this.params)
      getDataModuleList(this.params).then((res) => {
        const pagination = { ...this.pagination }
        console.log(res)
        pagination.total = res.result.total
        this.loading = false
        this.data = res.result.records
        this.pagination = pagination
      })
    },
    fetch(node) {
      this.params=this.$options.data().params
      this.pagination=this.$options.data().pagination
      if(node){
        this.params.folderId = node
      }else {
        this.params.folderId = 0;
      }
      this.loadData()
      // queryData({
      //   results: 10,
      //   ...params,
      // }).then(({ data }) => {
      //   const pagination = { ...this.pagination }
      //   // Read total count from server
      //   // pagination.total = data.totalCount;
      //   pagination.total = 200
      //   this.loading = false
      //   this.data = data.results
      //   this.pagination = pagination
      // })
    }
  }
}
</script>

<style scoped>
@import '~@assets/less/common.less'
</style>