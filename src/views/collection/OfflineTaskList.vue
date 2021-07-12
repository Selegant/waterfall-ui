<template>
  <a-card :bordered="false">
    <div v-if="showList">
      <!-- 操作按钮区域 -->
      <div class="table-operator" :md="24" :sm="24" style="margin: -25px 0px 10px 0px">
        <a-button @click="handleAdd" type="primary" icon="plus">新增离线任务</a-button>

        <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1" @click="batchDel">
              开启
            </a-menu-item>
            <a-menu-item key="2" @click="batchDel">
              停止
            </a-menu-item>
            <a-menu-item key="3" @click="batchDel">
              删除
            </a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px;margin-top: 1px"> 批量操作
            <a-icon type="down"/>
          </a-button>
        </a-dropdown>
      </div>

      <!-- table区域-begin -->
      <div>
        <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
          <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a
          style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
          <a style="margin-left: 24px" @click="onClearSelected">清空</a>
        </div>

        <a-table
          ref="table"
          size="middle"
          bordered
          rowKey="id"
          :columns="columns"
          :dataSource="dataSource"
          :pagination="ipagination"
          :loading="loading"
          :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
          @change="handleTableChange">
            <span slot="collectionType" slot-scope="collectionType">
              <template>
                <div style="">
                  <a-tag v-if="collectionType===0" color="#2db7f5">全量</a-tag>
                  <a-tag v-if="collectionType===1" color="#87d068">增量</a-tag>
                </div>
              </template>
            </span>
            <span slot="triggerStatus" slot-scope="triggerStatus">
              <template>
                <div style="">
                  <a-tag v-if="triggerStatus===0" color="#f50">停止</a-tag>
                  <a-tag v-if="triggerStatus===1" color="#87d068">运行</a-tag>
                </div>
              </template>
            </span>
            <span slot="timeFormat" slot-scope="time">
              <template>
               <span v-if="moment(time).isValid()">{{moment(time).format('YYYY-MM-DD HH:mm:ss')}}</span>
               <span v-else>—</span>
              </template>
            </span>
            <span slot="action" slot-scope="text, record">
              <a @click="handleEdit(record)">编辑</a>
              <a-divider type="vertical"/>
              <a-dropdown>
                <a class="ant-dropdown-link">
                  更多 <a-icon type="down"/>
                </a>
                <a-menu slot="overlay">
                   <a-menu-item>
                    <a href="javascript:;" @click="handleDetail(record)">执行一次</a>
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

    <!-- table区域-end -->
    <!-- 表单区域 -->
    <!--    <jeecgOrderCustomer-modal ref="modalForm" @ok="modalFormOk"></jeecgOrderCustomer-modal>-->
    <add-offline-task ref="add" @close="closeOfflineTask"></add-offline-task>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
import AddOfflineTask from './modules/AddOfflineTask'
import moment from 'moment'

export default {
  name: 'OfflineTaskList',
  mixins: [JeecgListMixin],
  components: {
    AddOfflineTask
  },
  data() {
    return {
      moment,
      showList: true,
      showAdd: false,
      // 表头
      columns: [
        {
          title: '编号',
          dataIndex: 'id'
        },
        {
          title: '离线任务名称',
          dataIndex: 'taskName'
        },
        {
          title: '源表',
          dataIndex: 'originalTable'
        },
        {
          title: '目标表',
          dataIndex: 'targetTable'
        },
        {
          title: '全量/增量',
          dataIndex: 'collectionType',
          scopedSlots: { customRender: 'collectionType' }
        },
        // {
        //   title: '更新时间',
        //   dataIndex: 'updateTime'
        // },
        {
          title: '状态',
          dataIndex: 'triggerStatus',
          scopedSlots: { customRender: 'triggerStatus' }
        },
        {
          title: '上次运行时间',
          dataIndex: 'triggerLastTime',
          scopedSlots: { customRender: 'timeFormat'}
        },
        {
          title: '下次运行时间',
          dataIndex: 'triggerNextTime',
          scopedSlots: { customRender: 'timeFormat'}
        },
        {
          title: '操作',
          // dataIndex: 'action',
          width: '120px',
          fixed: 'right',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/offlineTask/list',
        delete: '/test/order/deleteCustomer',
        deleteBatch: '/test/order/deleteBatchCustomer',
      }
    }
  },
  methods: {
    loadData(arg) {
      if (arg === 1) {
        this.ipagination.current = 1
      }
      //update-begin--Author:kangxiaolin  Date:20190905 for：[442]主子表分开维护，生成的代码子表的分页改为真实的分页--------------------
      var params = this.getQueryParams()
      getAction(this.url.list, {
        orderId: params.mainId, pageNo: this.ipagination.current,
        pageSize: this.ipagination.pageSize
      }).then((res) => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        } else {
          this.dataSource = null
        }
      })
      //update-end--Author:kangxiaolin  Date:20190905 for：[442]主子表分开维护，生成的代码子表的分页改为真实的分页--------------------

    },
    getOrderMain(orderId) {
      this.queryParam.mainId = orderId
      this.loadData(1)
    },
    handleAdd: function () {
      this.$refs.add.show()
    },
    closeOfflineTask() {
      // this.showAdd = false
      // this.showList = true
      this.$refs[`table${this.params.purpose}`][0].refresh()
    },
  }
}
</script>
<style scoped>
.ant-card {
  margin-left: -30px;
  margin-right: -30px;
}
</style>