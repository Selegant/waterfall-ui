<template>
  <div style="width:100%;">
    <!-- 查询区域 -->
    <div class="">
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
      params: {}
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
    fetch() {
      this.loading = true
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