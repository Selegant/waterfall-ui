<template>
  <!--  <a-card :bordered="false">-->
<!--  <div style="padding: 30px 30px 0px 30px;">-->
<!--    <div style="display: flex;">-->
      <a-row :gutter="20">
        <a-col :md="3" :sm="24">
          <div style="background: #FFFFFF;">
            <div style="display: flex;flex-direction:row-reverse;height: 25px;line-height:25px;background:#F5F5F5 ">
              <div>
                <i class="action-jeecg actionxinzengwenjianjia-icon" style="font-size: 20px;" @click="()=>{this.$refs.modelType.open()}"/>
              </div>
            </div>
            <div style="padding: 0px 15px;">
              <a-tree :tree-data="treeData" show-icon defaultExpandAll @select="changeTreeNode">
                <template #title="{ key: treeKey, title }">
                  <a-dropdown :trigger="['contextmenu']">
                    <span>{{ title }}</span>
                    <template #overlay>
                      <a-menu @click="({ key: menuKey }) => onContextMenuClick(treeKey, menuKey)">
                        <a-menu-item key="1">编辑</a-menu-item>
                        <a-menu-item key="2">删除</a-menu-item>
                      </a-menu>
                    </template>
                  </a-dropdown>
                </template>
                <a-icon slot="switcherIcon" type="caret-down" style="font-size: 15px;" />
                <i slot="wenjianjia" class="action-jeecg actionwenjianjia2" style="font-size: 20px;color: #0c8fcf" />
                <a-icon slot="smile" type="smile-o" />
              </a-tree>
            </div>
          </div>
        </a-col>
        <a-col :md="21" :sm="24">
          <div>
            <div class="card-container">
              <a-tabs defaultActiveKey="1" type="card">
                <a-tab-pane tab="模型设计" key="1">
                  <!--          <offline-task-list ref="OfflineTaskList"></offline-task-list>-->
<!--                  <div style="width:100%;height: calc(63.6vh)"></div>-->
                  <model-list ref="modelList"></model-list>
                </a-tab-pane>
                <a-tab-pane tab="已发布模型" key="2" forceRender>
                  <!--          <online-task-list ref="OnlineTaskList"></online-task-list>-->
                  <model-list></model-list>
                </a-tab-pane>
              </a-tabs>
            </div>
          </div>
        </a-col>
        <model-type ref="modelType" :treeData="modelTreeData" @refresh="init()"></model-type>
      </a-row>
<!--    </div>-->

<!--  </div>-->
  <!--  </a-card>-->
</template>

<script>
import ModelType from './modules/ModelType'
import ModelList from './modules/ModelList'
import { getModelFolder,deleteModelFolder } from '../../api/api'
import { getFolderItem } from "@/utils/util"
// const treeData = [
//   {
//     title: 'parent 1',
//     key: '0-0',
//     slots: {
//       icon: 'wenjiajia',
//     },
//     children: [
//       { title: 'leaf2', key: '0-0-0', slots: { icon: 'wenjiajia' } },
//       { title: 'leaf2', key: '0-0-1', slots: { icon: 'wenjiajia' } },
//     ],
//   },
// ]

export default {
  name: 'DataModel',
  components: { ModelType, ModelList },
  data() {
    return {
      treeData: [{
        title: '全部',
        key: '0',
        slots: {
          icon: 'wenjianjia'
        },
        value: '0',
        children:[]
      }],
      modelTreeData:[{
        title: '全部',
        key: '0',
        slots: {
          icon: 'wenjianjia'
        },
        value: '0',
        children:[]
      }]
    }
  },
  mounted() {
    this.init()
  },
  methods:{
    init(){
      getModelFolder().then((res)=>{
        this.treeData[0].children = res.result
      })
      getModelFolder().then((res)=>{
        const result = res.result
        var tree = []
        for (let resKey in result) {
          tree.push(result[resKey])
        }
        for (let resKey in tree) {
          tree[resKey].children = []
        }
        this.modelTreeData[0].children = tree
      })
    },
    changeTreeNode(selectedKeys){
      console.log(selectedKeys)
      this.$refs.modelList.fetch(selectedKeys[0])
    },
    onContextMenuClick(treeKey, menuKey){
      console.log(`treeKey: ${treeKey}, menuKey: ${menuKey}`);
      if(menuKey == '1'){
        // 编辑
        let folderItem = getFolderItem('key',treeKey,this.treeData)
        let {parentId,remark,title} = {...folderItem}
        this.$refs.modelType.open({id:treeKey,folderName:title,remark,parentId})
      }else{
        // 删除
        deleteModelFolder({id:treeKey}).then(async (res) =>{
              if (res.success) {
                await this.init()
                await this.$refs.modelList.fetch()
                this.$message.success('删除成功')
              } else {
                this.$message.error(res.message)
              }
            })
      }
    }
  }
}
</script>

<style>

.card-container {
  background: #f5f5f5;
  overflow: hidden;
  /*padding: 24px;*/
  padding: 0px !important;
}

.card-container > .ant-tabs-card > .ant-tabs-content {
  /*height: 120px;*/
  /*margin-top: -16px;*/
}

.card-container > .ant-tabs-card > .ant-tabs-content > .ant-tabs-tabpane {
  background: #fff;
  /*padding: 16px;*/
}

.card-container > .ant-tabs-card > .ant-tabs-bar {
  border-color: #fff;
}

.card-container > .ant-tabs-card > .ant-tabs-bar .ant-tabs-tab {
  border-color: transparent;
  background: transparent;
}

.card-container > .ant-tabs-card > .ant-tabs-bar .ant-tabs-tab-active {
  border-color: #fff;
  background: #000;
}

.card-container .ant-tabs .ant-tabs-bar .ant-tabs-nav-container .ant-tabs-tab-active.ant-tabs-tab {
  border-top: #0c8fcf 2px solid !important;
}

</style>