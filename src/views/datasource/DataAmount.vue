<template>
  <a-card :bordered="false" class="card-area" style="height: 80%">
    <a-tabs default-active-key="1" @change="callback" tab-position="top">
      <a-tab-pane v-for="(index,item) in TABS" :key="item+1" :tab="`${index}`" force-render>
        <div style="display: flex;">
          <div style="margin-bottom: 20px;width: 200px;border-right: #E7E7E7 2px solid">
            <a-tree :tree-data="treeData" show-icon default-expand-all @select="getTablesAndViews">
              <div slot="MySQL">
                <i class="action-jeecg actionmy-SQL" style="color: #00A0E9;margin-right: 10px;" />
              </div>
              <div slot="Oracle">
                <i class="action-jeecg actionconnection-Oracle"  style="color: red;margin-right: 10px;"/>
              </div>
              <div slot="table">
                <i class="action-jeecg actiontable" style="margin-right: 10px;" />
              </div>
              <div slot="view">
                <i class="action-jeecg actiontable_viewer_old" style="margin-right: 10px;"/>
              </div>
            </a-tree>
          </div>
          <div style="width: 1300px;margin-left: 20px;">
            <div style="margin-bottom: 20px;">
              <a-row type="flex" justify="end">
                <a-button icon="sync" @click="asyncUpdateAmount" :loading="syncLoading">同步</a-button>
              </a-row>
            </div>
            <div>
              <s-table
                :ref="`table${item+1}`"
                size="default"
                :columns="columns"
                :data="loadData"
                :showPagination = "false"
                bordered
              >
                <span slot="requiredTime" slot-scope="requiredTime">
                  <template>
                    <div style="">
                      <p>{{ moment.duration(requiredTime).asSeconds() }}(秒)</p>
                    </div>
                  </template>
                </span>
              </s-table>
            </div>
          </div>
        </div>
      </a-tab-pane>
    </a-tabs>



    <data-source-modal ref="modalForm" @ok="init"></data-source-modal>
  </a-card>
</template>

<script>
import STable from '@/components/table/'
import { getDataSourceTreeList, getDataSourceTables,getAmountList,asyncAmount } from '@/api/api'
import DataSourceModal from './modules/DataSourceModal'
// import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getServiceList } from '@/api/manage'
import moment from 'moment'

const MYSQL = 'MySQL'
const ORACLE = 'Oracle'

const TABS = ['数据端','目标端']

export default {
  name: 'DataAmount',
  // mixins: [JeecgListMixin],
  components: {
    DataSourceModal,
    STable
  },
  data() {
    return {
      moment,
      MYSQL,
      ORACLE,
      TABS,
      params: {
        'purpose': 1,
        'id': '-1',
        'type': '1'
      },
      isLeaf: false,
      syncLoading: false,
      treeData: [],
      columns: [
        {
          title: '编号',
          dataIndex: 'id'
        },
        {
          title: '表名',
          dataIndex: 'tableName'
        },
        {
          title: '数据量',
          sorter: (a,b) => a.amount-b.amount,
          dataIndex: 'amount'
        },
        {
          title: '更新时间',
          dataIndex: 'updateTime',
          sorter: (a,b) => moment(a.updateTime).diff(moment(b.updateTime),'seconds'),
          // needTotal: true
        },
        {
          title: '更新所用时间',
          dataIndex: 'requiredTime',
          sorter: (a,b) => a.requiredTime-b.requiredTime,
          scopedSlots: { customRender: 'requiredTime' }
        }
      ],
      loadData: parameter => {
        return getAmountList(this.params.id,this.params.type)
          .then(res => {
            console.log(res.result)
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
      this.params.purpose = key
      getDataSourceTreeList({ 'purpose': key }).then((res) => {
        if (res.success) {
          this.treeData = res.result.trees
        }
      })
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
    getTablesAndViews(e){
      console.log(`table${this.params.purpose}`)
      if(e[0].indexOf('-')===-1){
        this.isLeaf = false
        return
      }
      this.isLeaf = true
      this.params.id = e[0].split('-')[0];
      this.params.type = e[0].split('-')[1];
      this.$refs[`table${this.params.purpose}`][0].refresh()
    },
    asyncUpdateAmount(){
      if(!this.isLeaf){
        this.$message.warning('请选择具体的数据源所对应的表和视图！')
        return
      }
      this.syncLoading = true
      const that = this
      asyncAmount(this.params.id,this.params.type)
        .then(res => {
          that.syncLoading = false
          that.$refs[`table${this.params.purpose}`][0].refresh()
          return res.result
        })
    }
  }
}
</script>

<style scoped>

</style>