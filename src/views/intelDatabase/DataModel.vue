<template>
  <a-card :bordered="false">
    <div>
      <div style="display: flex">
        <div>
          <div style="display: flex;height: 25px;line-height:25px;background: #00A0E9;color: #FFFFFF">
            <div><a-icon type="folder-add" style="font-size: 20px" /></div>
          </div>
          <a-directory-tree multiple default-expand-all @select="onSelect" @expand="onExpand">
            <a-tree-node key="0-0" title="parent 0">
              <a-tree-node key="0-0-0" title="leaf 0-0" is-leaf/>
              <a-tree-node key="0-0-1" title="leaf 0-1" is-leaf/>
            </a-tree-node>
            <a-tree-node key="0-1" title="parent 1">
              <a-tree-node key="0-1-0" title="leaf 1-0" is-leaf/>
              <a-tree-node key="0-1-1" title="leaf 1-1" is-leaf/>
            </a-tree-node>
          </a-directory-tree>
        </div>
        <div>
          <a-tabs defaultActiveKey="1">
            <a-tab-pane tab="模型设计" key="1">
              <!--          <offline-task-list ref="OfflineTaskList"></offline-task-list>-->
            </a-tab-pane>
            <a-tab-pane tab="已发布模型" key="2" forceRender>
              <!--          <online-task-list ref="OnlineTaskList"></online-task-list>-->
            </a-tab-pane>
          </a-tabs>
        </div>
      </div>
    </div>
  </a-card>
</template>

<script>
export default {
  name: 'DataModel'
}
</script>

<style scoped>

</style>