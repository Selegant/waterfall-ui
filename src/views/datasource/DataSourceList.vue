<template>
  <a-card :bordered="false" class="card-area" style="height: 80%">
    <a-tabs default-active-key="1" @change="callback" tab-position="left">
      <a-tab-pane key="1" tab="数据端">
        <div style="margin-bottom: 20px">
          <a-card style="width: 300px;">
            <div style="text-align: center">
              <a-button  icon="plus" :size="large" block style="border: none">新增数据源</a-button>
            </div>
          </a-card>
        </div>
        <a-list :grid="{ gutter: 16, xs: 1, sm: 2, md: 4, lg: 4, xl: 6, xxl: 4 }" :data-source="data">
          <a-list-item slot="renderItem" slot-scope="item, index">
            <a-card :title="item.title">
              <label>类型：</label><a-tag color="green">{{item.type}}</a-tag><br/>
              <label>IP：</label><span>{{item.ip}}</span><br/>
              <label>端口号：</label><span>{{item.port}}</span><br/>
              <label>用户名：</label><span>{{item.username}}</span><br/>
              <label>数据库名：</label><span>{{item.databaseName}}</span><br/>
              <div style="text-align: right">
                <div>
                  <a-button type="primary" icon="edit" size="small">编辑</a-button>
                  &nbsp;
                  <a-button type="danger" icon="delete" size="small">删除</a-button>
                </div>
              </div>t
            </a-card>
          </a-list-item>
        </a-list>
      </a-tab-pane>
      <a-tab-pane key="2" tab="目标端" force-render>
        <div style="margin-bottom: 20px">
          <a-card style="width: 300px;">
            <div style="text-align: center">
              <a-button  icon="plus" :size="large" block style="border: none">新增数据源</a-button>
            </div>
          </a-card>
        </div>
        <a-list :grid="{ gutter: 16, xs: 1, sm: 2, md: 4, lg: 4, xl: 6, xxl: 4 }" :data-source="data">
          <a-list-item slot="renderItem" slot-scope="item, index">
            <a-card :title="item.title">
              <label>类型：</label><a-tag color="green">{{item.type}}</a-tag><br/>
              <label>IP：</label><span>{{item.ip}}</span><br/>
              <label>端口号：</label><span>{{item.port}}</span><br/>
              <label>用户名：</label><span>{{item.username}}</span><br/>
              <label>数据库名：</label><span>{{item.databaseName}}</span><br/>
              <div style="text-align: right">
                <div>
                  <a-button type="primary" icon="edit" size="small">编辑</a-button>
                  &nbsp;
                  <a-button type="danger" icon="delete" size="small">删除</a-button>
                </div>
              </div>
            </a-card>
          </a-list-item>
        </a-list>
      </a-tab-pane>
    </a-tabs>
  </a-card>
</template>

<script>

const data = [
  {
    title: 'DB1',
    type: 'MySQL',
    ip: 'selegant.synology.me',
    port: '33306',
    username: 'root',
    databaseName: 'waterfall'
  },
  {
    title: 'DB2',
    type: 'MySQL',
    ip: 'selegant.synology.me',
    port: '33306',
    username: 'root',
    databaseName: 'waterfall'
  },
  {
    title: 'DB3',
    type: 'MySQL',
    ip: 'selegant.synology.me',
    port: '33306',
    username: 'root',
    databaseName: 'waterfall'
  },
  {
    title: 'Title 4',
    type: 'MySQL',
    ip: 'selegant.synology.me',
    port: '33306',
    username: 'root',
    databaseName: 'waterfall'
  },
  {
    title: 'Title 5',
    type: 'MySQL',
    ip: 'selegant.synology.me',
    port: '33306',
    username: 'root',
    databaseName: 'waterfall'
  },
  {
    title: 'Title 6',
    type: 'MySQL',
    ip: 'selegant.synology.me',
    port: '33306',
    username: 'root',
    databaseName: 'waterfall'
  },
];

export default {
  name: 'index',
  data() {
    return {
      data
    };
  },
  methods: {
    callback(key) {
      console.log(key);
    },
  }
}
</script>

<style scoped>

</style>