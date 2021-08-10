<template>
  <div>
    <a-modal title="创建表" :visible="showModal" @cancel="()=>{this.showModal=false}" :width="1000" @ok="createTable" :confirm-loading="loading" >
      <a-card :bordered="false" class="card-area" style="height: 80%">
<!--        <a-table :columns="columns" :data-source="data" :pagination="false">-->
<!--          <a slot="name" slot-scope="text">{{ text }}</a>-->
<!--        </a-table>-->
<!--        <p style="font-weight: bold">表名: {{task.originalTable}}</p>-->
<!--        <br/>-->
        <div style="height: 400px;overflow-y: auto;" >
          <div class="table-box" style="background: #EFF2F5">
            <div>字段名</div>
            <div>源数据类型</div>
            <div>当前数据类型</div>
            <div>备注</div>
          </div>
          <div v-for="(item,index) in data" :key="index" class="table-box">
            <div>{{ item.sourceColumnName }}</div>
            <div>{{ item.sourceColumnType }}</div>
            <div>
              <a-select v-model="item.targetColumnType">
                <a-select-option v-for="(type,index) in dataTypes" :key="index">
                  {{type}}
                </a-select-option>
              </a-select>
            </div>
            <div>
              <a-input v-model="item.columnComment"></a-input>
            </div>
          </div>
        </div>

      </a-card>
    </a-modal>
  </div>
</template>

<script>
import { createHiveTable, getDataTypes, getTargetTypeColumns } from '../../../api/api'

export default {
  name: 'CreateTable',
  data() {
    return {
      loading: false,
      task:'',
      data:[],
      dataTypes:[],
      columns:[
        {
          title: '字段名',
          dataIndex: 'sourceColumnName',
          key: 'sourceColumnName',
          width: 150
        },
        {
          title: '源数据类型',
          dataIndex: 'sourceColumnType',
          key: 'sourceColumnType',
          width: 150,
        },
        {
          title: '当前数据类型',
          dataIndex: 'targetColumnType',
          key: 'targetColumnType',
          width: 150
        },
        {
          title: '备注',
          dataIndex: 'targetColumnType',
          key: 'targetColumnType',
          width: 200,
        }
      ],
      showModal: false
    };
  },
  methods:{
    init(task){
      this.showModal=true
      console.log(task)
      this.task = task
      getDataTypes('HIVE').then((res)=>{
        this.dataTypes=res.result
      })
      getTargetTypeColumns(task.originalId,task.targetId,task.originalTable).then((res)=>{
        this.data = res.result
      })
    },
    createTable(){
      const params = {
        dbId:this.task.targetId,
        tableName:this.task.originalTable,
        columns: this.data
      }
      const that = this
      this.loading = true
      // this.showModal=false
      // this.$emit('addTable',that.task.originalTable)
      createHiveTable(params).then((res)=>{
        if(res.code===200){
          this.loading = false
          this.$message.success("创建成功")
          this.showModal=false
          this.$emit('addTable',that.task.originalTable)
        }else {
          this.$message.success("创建失败")
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.table-box {
  display: flex;line-height: 50px;justify-content:space-around;
  div {
    width: 150px;
  }
}
</style>