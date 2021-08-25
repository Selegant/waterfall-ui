<template>
  <div style="width:100%;">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <!-- 搜索区域 -->
      <a-form-model layout="inline" :model="params" >
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
              <a-button type="primary" icon="search" style="margin-left: 21px">查询</a-button>
              <a-button icon="reload" style="margin-left: 8px">重置</a-button>
            </a-col>
          </span>

          <span style="display: flex;justify-content: flex-end" class="">
            <a-col>
              <a-button icon="vertical-align-bottom" style="margin-left: 21px">导入</a-button>
              <a-button type="primary"  icon="plus" style="margin-left: 8px">新建数据模型</a-button>
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
              <a @click="handleEdit(record)">编辑</a>
              <a-divider type="vertical"/>
              <a @click="handleEdit(record)">质量</a>
              <a-divider type="vertical"/>
              <a @click="handleEdit(record)">发布</a>
              <a-divider type="vertical"/>
              <a-dropdown>
                <a class="ant-dropdown-link">
                  更多 <a-icon type="down"/>
                </a>
                <a-menu slot="overlay">
                   <a-menu-item>
                    <a href="javascript:;" @click="triggerTask(record)">执行一次</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a href="javascript:;" @click="handleDetail(record)">开启</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a href="javascript:;" @click="handleDetail(record)">停止</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a href="javascript:;" @click="handleDetail(record)">详情</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                      <a>删除</a>
                    </a-popconfirm>
                  </a-menu-item>
                </a-menu>
              </a-dropdown>
            </span>
      </a-table>
    </div>

  </div>
</template>

<script>
import { getDataModuleList } from '../../../api/api'

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
    dataIndex: 'ramark',
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
    width: '150px'
  },
]

export default {
  name: 'ModelList',
  data() {
    return {
      data: [],
      pagination: {
        current: 1,
        pageSize: 10,
        pageSizeOptions: ['10', '20', '30'],
        showTotal: (total, range) => {
          // return range[0] + "-" + range[1] + " 共" + total + "条"
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
    this.fetch()
  },
  methods: {
    handleTableChange(pagination, filters, sorter) {
      console.log(pagination)
      const pager = { ...this.pagination }
      pager.current = pagination.current
      this.pagination = pager
      this.params ={
        pageNo:pager.current
      }
      this.fetch()
    },
    fetch(node) {
      this.params.modelName=''
      this.loading = true
      if(node){
        this.params.folderId = node
      }else {
        this.params.folderId = 0;
      }
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