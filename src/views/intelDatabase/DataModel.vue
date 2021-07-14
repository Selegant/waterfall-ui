<template>
  <!--  <a-card :bordered="false">-->
  <div style="padding: 30px 30px 0px 30px;">
    <div style="display: flex;">
      <div style="width:  calc(30vh);background: #FFFFFF;height: calc(72vh);">
        <div style="display: flex;flex-direction:row-reverse;height: 25px;line-height:25px;background:#EFF2F5 ">
          <div>
            <i class="action-jeecg actionxinzengwenjianjia-icon" style="font-size: 20px;"/>
          </div>
        </div>
        <div style="padding: 0px 15px;">
          <a-tree :tree-data="treeData" show-icon  default-expand-all>
            <a-icon slot="switcherIcon" type="caret-down" style="font-size: 15px;" />
            <i slot="wenjiajia" class="action-jeecg actionwenjianjia2" style="font-size: 20px;color: #0c8fcf" />
            <a-icon slot="smile" type="smile-o" />
          </a-tree>
        </div>

      </div>
      <div style="margin-left: 20px;">
        <div class="card-container">
          <a-tabs defaultActiveKey="1" type="card">
            <a-tab-pane tab="模型设计" key="1">
              <!--          <offline-task-list ref="OfflineTaskList"></offline-task-list>-->
              <div style="width:calc(160vh);height: calc(63.6vh)"></div>
            </a-tab-pane>
            <a-tab-pane tab="已发布模型" key="2" forceRender>
              <!--          <online-task-list ref="OnlineTaskList"></online-task-list>-->
              <div style="width:calc(160vh);height: calc(63.6vh)"></div>
            </a-tab-pane>
          </a-tabs>
        </div>
      </div>
    </div>
  </div>
  <!--  </a-card>-->
</template>

<script>
const treeData = [
  {
    title: 'parent 1',
    key: '0-0',
    slots: {
      icon: 'wenjiajia',
    },
    children: [
      { title: 'leaf2', key: '0-0-0', slots: { icon: 'wenjiajia' } },
      { title: 'leaf2', key: '0-0-1', slots: { icon: 'wenjiajia' } },
    ],
  },
]

export default {
  name: 'DataModel',
  data() {
    return {
      treeData
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