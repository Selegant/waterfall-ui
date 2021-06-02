<template>
  <a-card :bordered="false" class="card-area" style="height: 80%">
    <a-tabs default-active-key="1" @change="callback" tab-position="top">
      <a-tab-pane key="1" tab="数据端">
        <div style="display: flex;">
          <div style="margin-bottom: 20px;width: 200px;border-right: #E7E7E7 2px solid">
<!--            <a-menu style="width: 256px"-->
<!--                    mode="inline" @click="">-->
<!--              <a-menu-item v-for="(item,index) in data" :key="item.id" :value="item.id">-->
<!--                &lt;!&ndash;              <a-icon type="calendar" />&ndash;&gt;-->
<!--                <a-tag v-if="item.dbType==MYSQL" color="#2db7f5">-->
<!--                  <div style="width: 40px">{{ item.dbType }}</div>-->
<!--                </a-tag>-->
<!--                <a-tag v-if="item.dbType==ORACLE" color="#f50">-->
<!--                  <div style="width: 40px">{{ item.dbType }}</div>-->
<!--                </a-tag>-->
<!--                {{ item.dataSourceName }}-->
<!--              </a-menu-item>-->
<!--            </a-menu>-->
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
                ref="table"
                size="default"
                :columns="columns"
                :data="loadData"
                :showPagination = "false"
                bordered
              >
              </s-table>
            </div>
          </div>
        </div>
      </a-tab-pane>


      <a-tab-pane key="2" tab="目标端" force-render>
<!--        <div style="margin-bottom: 20px">-->
<!--          <a-menu style="width: 256px"-->
<!--                  mode="inline" @click="">-->
<!--            <a-menu-item v-for="(item,index) in data" :key="item.id" :value="item.id">-->
<!--              &lt;!&ndash;              <a-icon type="calendar" />&ndash;&gt;-->
<!--              <a-tag v-if="item.dbType==MYSQL" color="#2db7f5">-->
<!--                <div style="width: 40px">{{ item.dbType }}</div>-->
<!--              </a-tag>-->
<!--              <a-tag v-if="item.dbType==ORACLE" color="#f50">-->
<!--                <div style="width: 40px">{{ item.dbType }}</div>-->
<!--              </a-tag>-->
<!--              {{ item.dataSourceName }}-->
<!--            </a-menu-item>-->
<!--          </a-menu>-->
<!--        </div>-->
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

const MYSQL = 'MySQL'
const ORACLE = 'Oracle'

export default {
  name: 'DataAmount',
  // mixins: [JeecgListMixin],
  components: {
    DataSourceModal,
    STable
  },
  data() {
    return {
      MYSQL,
      ORACLE,
      params: {
        'purpose': 1,
        'id': '-1'
      },
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
          dataIndex: 'amount'
        },
        {
          title: '更新时间',
          dataIndex: 'updateTime'
          // needTotal: true
        },
        {
          title: '更新所用时间',
          dataIndex: 'requiredTime',
          sorter: true
        }
      ],
      loadData: parameter => {
        return getAmountList(this.params.id)
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
      console.log(e[0])
      this.params.id = e[0].substring('-')[0];
      this.$refs.table.refresh()
    },
    asyncUpdateAmount(){
      this.syncLoading = true
      asyncAmount(this.params.id)
        .then(res => {
          this.syncLoading = false
          this.$refs.table.refresh()
          return res.result
        })
    }
  }
}
</script>

<style scoped>

</style>