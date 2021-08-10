<template xmlns="http://www.w3.org/1999/html">
  <a-card :bordered="false" class="card-area" style="height: 80%">
    <a-tabs default-active-key="1" @change="callback" tab-position="left">
      <a-tab-pane key="1" tab="数据端">
        <div style="margin-bottom: 20px">
          <a-card style="width: 300px;">
            <div style="text-align: center">
              <a-button @click="handleAdd(params)" icon="plus" :size="large" type="link">新增数据源</a-button>
            </div>
          </a-card>
        </div>
        <a-list :grid="{ gutter: 16, xs: 1, sm: 2, md: 4, lg: 4, xl: 6, xxl: 3 }" :data-source="data">
          <a-list-item slot="renderItem" slot-scope="item, index">
            <a-card :title="item.dataSourceName">
              <label>数据库类型：</label>
              <a-tag v-if="item.dbType==MYSQL" color="#2db7f5"><div style="width: 50px;text-align: center;">{{item.dbType}}</div></a-tag>
              <a-tag v-if="item.dbType==ORACLE" color="#f50"><div style="width: 50px;text-align: center;">{{item.dbType}}</div></a-tag>
              <a-tag v-if="item.dbType==HIVE" color="#FCE92F"><div style="width: 50px;text-align: center;color:#333333">{{item.dbType}}</div></a-tag>
              <br/>
              <label>Host：</label><span>{{item.host}}</span><br/>
              <label>Port：</label><span>{{item.port}}</span><br/>
              <label>URL：</label><span>{{item.jdbcUrl}}</span><br/>
<!--              <label>端口号：</label><span>{{item.port}}</span><br/>-->
              <label>用户名：</label><span>{{item.username}}</span><br/>
<!--              <label>数据库名：</label><span>{{item.databaseName}}</span><br/>-->
              <div style="text-align: right">
                <div>
                  <a-button type="primary" icon="edit" size="small" @click="handleEdit(item)">编辑</a-button>
                  &nbsp;
                  <a-popconfirm title="确定删除数据源？" @confirm="handleDelete(item)" okText="确定" cancelText="取消">
                    <a-button type="danger" icon="delete" size="small">删除</a-button>
                  </a-popconfirm>
                </div>
             </div>
            </a-card>
          </a-list-item>
        </a-list>
      </a-tab-pane>
      <a-tab-pane key="2" tab="目标端" force-render>
        <div style="margin-bottom: 20px">
          <a-card style="width: 300px;">
            <div style="text-align: center">
              <a-button @click="handleAdd(params)" icon="plus" :size="large" type="link">新增数据源</a-button>
            </div>
          </a-card>
        </div>
        <a-list :grid="{ gutter: 16, xs: 1, sm: 2, md: 4, lg: 4, xl: 6, xxl: 3 }" :data-source="data">
          <a-list-item slot="renderItem" slot-scope="item, index">
            <a-card :title="item.dataSourceName">
              <label>数据库类型：</label>
              <a-tag v-if="item.dbType==MYSQL" color="#2db7f5"><div style="width: 40px;text-align: center">{{item.dbType}}</div></a-tag>
              <a-tag v-if="item.dbType==ORACLE" color="#f50"><div style="width: 40px;text-align: center">{{item.dbType}}</div></a-tag>
              <a-tag v-if="item.dbType==HIVE" color="#FCE92F"><div style="width: 40px;text-align: center;color:#333333">{{item.dbType}}</div></a-tag>
              <br/>
              <label>Host：</label><span>{{item.host}}</span><br/>
              <label>Port：</label><span>{{item.port}}</span><br/>
              <label>URL：</label><span>{{item.jdbcUrl}}</span><br/>
              <!--              <label>端口号：</label><span>{{item.port}}</span><br/>-->
              <label>用户名：</label><span>{{item.username}}</span><br/>
              <!--              <label>数据库名：</label><span>{{item.databaseName}}</span><br/>-->
              <div style="text-align: right">
                <div>
                  <a-button type="primary" icon="edit" size="small" @click="handleEdit(item)">编辑</a-button>
                  &nbsp;
                  <a-popconfirm title="确定删除数据源？" @confirm="handleDelete(item)" okText="确定" cancelText="取消">
                    <a-button type="danger" icon="delete" size="small">删除</a-button>
                  </a-popconfirm>
                </div>
              </div>
            </a-card>
          </a-list-item>
        </a-list>
      </a-tab-pane>
    </a-tabs>

    <data-source-modal ref="modalForm" @ok="init"></data-source-modal>
  </a-card>
</template>

<script>
import {getDataSourceList,delDataSource} from '@/api/api'
import DataSourceModal from './modules/DataSourceModal'
import {JeecgListMixin} from '@/mixins/JeecgListMixin'
const MYSQL='MYSQL'
const ORACLE = 'ORACLE'
const HIVE = 'HIVE'
export default {
  name: 'DataSourceList',
  mixins: [JeecgListMixin],
  components: {
    DataSourceModal,
  },
  data() {
    return {
      MYSQL,
      ORACLE,
      HIVE,
      params:{
        'purpose': 1
      },
      data:[]
    };
  },
  created() {
    this.init();
  },
  methods: {
    callback(key) {
      this.params.purpose = key
      getDataSourceList({ 'purpose':key }).then((res)=>{
        if(res.success){
          this.data = res.result
        }
      });
    },
    init(){
      getDataSourceList({ 'purpose':this.params.purpose }).then((res)=>{
        if(res.success){
          this.data = res.result
        }
      });
    },
    handleDelete(item){
      const that = this;
      delDataSource([item.id]).then((res)=>{
        if(res.success){
          that.data = res.result
          that.init();
        }
      });
    }
  }
}
</script>

<style scoped>

</style>