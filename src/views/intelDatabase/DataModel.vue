<template>
  <!--  <a-card :bordered="false">-->
<!--  <div style="padding: 30px 30px 0px 30px;">-->
<!--    <div style="display: flex;">-->
      <a-row :gutter="20">
        <a-col :md="3" :sm="24">
          <div style="background: #FFFFFF;height: calc(72vh);">
            <div style="display: flex;flex-direction:row-reverse;height: 25px;line-height:25px;background:#F5F5F5 ">
              <div>
                <i class="action-jeecg actionxinzengwenjianjia-icon" style="font-size: 20px;" @click="()=>{this.$refs.modelType.open()}"/>
              </div>
            </div>
            <div style="padding: 0px 15px;">
              <a-tree :tree-data="treeData" show-icon  default-expand-all>
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
                  <div style="width:100%;height: calc(63.6vh)"></div>
                </a-tab-pane>
                <a-tab-pane tab="已发布模型" key="2" forceRender>
                  <!--          <online-task-list ref="OnlineTaskList"></online-task-list>-->
                  <div style="width:100%;height: calc(63.6vh)"></div>
                </a-tab-pane>
              </a-tabs>
            </div>
          </div>
        </a-col>
        <model-type ref="modelType" :treeData="treeData"></model-type>
      </a-row>
<!--    </div>-->

<!--  </div>-->
  <!--  </a-card>-->
</template>

<script>
import ModelType from './modules/ModelType'
import { getModelFolder } from '../../api/api'
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
  components: { ModelType },
  data() {
    return {
      treeData: [{
        title: '全部',
        key: '0',
        slots: {
          icon: 'wenjianjia'
        },
        value: 'ALL',
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