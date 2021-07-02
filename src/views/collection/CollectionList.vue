<template>
  <div>
    <a-card :bordered="false" class="card-area" style="height: 80%">
      <a-tabs default-active-key="1" @change="callback" tab-position="left" v-show="showList">
        <a-tab-pane v-for="(index,item) in TABS" :key="`${item+1}`" :tab="`${index}`" force-render>
          <div style="display: flex;">
            <div style="width: 1500px;">
              <div style="margin-bottom: 20px;">
                <a-row type="flex">
                  <a-col>
                    <a-button icon="plus" type="primary" @click="addOfflineTask">新增任务</a-button>
                  </a-col>
                  <a-col>
                    <a-input-search :allowClear="true" v-model="queryParam.queryParam" placeholder="输入任务名或表名搜索" style="margin-left: 30px;width: 300px" @search="searchTaskList"></a-input-search>
                  </a-col>
                </a-row>
              </div>
              <div>
                <s-table
                  :ref="`table${item+1}`"
                  size="default"
                  :columns="columns"
                  :data="loadData"
                  bordered
                  :pagination="true"
                  v-if="params.purpose===1"
                >
                  <span slot="collectionType" slot-scope="collectionType">
                    <template>
                      <div style="text-align: center">
                        <a-tag v-if="collectionType===1" color="#2db7f5">全量</a-tag>
                        <a-tag v-if="collectionType===2" color="#87d068">增量</a-tag>
                      </div>
                    </template>
                  </span>
                  <span slot="action" slot-scope="record">
                    <template>
                      <a-button type="primary" size="small" icon="form" >编辑</a-button>
                      <a-divider type="vertical"/>
                      <a-button type="danger" size="small" icon="delete">删除</a-button>
                    </template>
                  </span>
                </s-table>
              </div>
            </div>
          </div>
        </a-tab-pane>
      </a-tabs>

      <add-offline-task ref="add" v-show="showAdd" @close="closeOfflineTask"></add-offline-task>
    </a-card>

  </div>
</template>

<script>
import STable from '@/components/table/'
import { getDataSourceTreeList, getDataSourceTables, getAmountList, asyncAmount, getOfflineTaskList } from '@/api/api'
import DataSourceModal from '../datasource/modules/DataSourceModal'
import AddOfflineTask from './modules/AddOfflineTask'

const MYSQL = 'MySQL'
const ORACLE = 'Oracle'

const TABS = ['离线任务', '实时任务']

export default {
  name: 'CollectionList',
  components: {
    DataSourceModal,
    STable,
    AddOfflineTask
  },
  data() {
    return {
      MYSQL,
      ORACLE,
      TABS,
      params: {
        'purpose': 1,
        'id': '-1',
        'type': '1'
      },
      queryParam:{},
      syncLoading: false,
      showList: true,
      showAdd: false,
      treeData: [],
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
        {
          title: '更新时间',
          dataIndex: 'updateTime'
        },
        {
          title: '状态',
          dataIndex: 'dbType'
        },
        {
          title: '上次运行时间',
          dataIndex: 'createTime',
        },
        {
          title: '操作',
          // dataIndex: 'action',
          width: '250px',
          scopedSlots: { customRender: 'action' }
        }
      ],
      loadData: parameter => {
        console.log(Object.assign(parameter,this.queryParam))
        return getOfflineTaskList(Object.assign(parameter,this.queryParam))
          .then(res => {
            // console.log(res.result)
            return res.result
          })
      }
    }
  },
  created() {
    this.init()
  },
  methods: {
    callback(key) {
      this.params.purpose = Number(key)
      // console.log(`table${key}`)
      // if(this.params.purpose ===1 ){
      //   this.$refs[`table${key}`][0].refresh()
      // }
    },
    init() {
      console.log(TABS[0].name)
      getDataSourceTreeList({ 'purpose': this.params.purpose }).then((res) => {
        if (res.success) {
          this.treeData = res.result.trees
        }
      })
    },
    handleDelete(item) {
      const that = this
      delDataSource([item.id]).then((res) => {
        if (res.success) {
          that.treeData = res.result
          that.init()
        }
      })
    },
    getTablesAndViews(e) {
      console.log(`table${this.params.purpose}`)
      this.params.id = e[0].split('-')[0]
      this.params.type = e[0].split('-')[1]
      this.$refs[`table${this.params.purpose}`][0].refresh()
    },
    addOfflineTask() {
      this.showAdd = true
      this.showList = false
    },
    closeOfflineTask(){
      this.showAdd = false
      this.showList = true
      this.$refs[`table${this.params.purpose}`][0].refresh()
    },
    searchTaskList(){
      this.$refs[`table${this.params.purpose}`][0].refresh()
    }
  }
}


</script>

<style scoped>

</style>