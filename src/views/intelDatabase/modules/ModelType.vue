<template>
  <div>
    <a-modal :visible="visible" title="新建模型层级" :width="600">
      <a-row :gutter="20">
        <a-col :md="24" :sm="24">
          <a-form :form="form" :label-col="{ span: 7 }" :wrapper-col="{ span: 14 }">
            <a-form-item label="上级目录">
              <w-tree-select
                v-decorator="['parentId', { rules: [{ required: true, message: '请选择菜单' }] }]"
                placeholder="请选择菜单"
                :tree-init-data="treeData">
              </w-tree-select>
            </a-form-item>
            <a-form-item label="层级名称">
              <a-input
                v-decorator="['folderName', { rules: [{ required: true, message: '请输入层级名称' }] }]"
              />
            </a-form-item>
            <a-form-item label="模型类型">
              <a-select
                v-decorator="['mark', { rules: [{ required: true, message: '请选则模型类型' }] }]"
              >
                <a-select-option value="1">原始数据STG</a-select-option>
                <a-select-option value="2">基础数据ODS</a-select-option>
                <a-select-option value="3">明细数据DWD</a-select-option>
                <a-select-option value="4">融合数据DWS</a-select-option>
                <a-select-option value="5">应用数据ADM</a-select-option>
                <a-select-option value="6">维度数据DIM</a-select-option>
                <a-select-option value="7">质量评估EVL</a-select-option>
                <a-select-option value="8">其他</a-select-option>
              </a-select>
            </a-form-item>
            <a-form-item label="描述">
              <a-input
                v-decorator="['folderName', { rules: [{ required: false, message: '请输入描述' }] }]"
              />
            </a-form-item>
          </a-form>
        </a-col>
      </a-row>
    </a-modal>
  </div>
</template>

<script>
import WTreeSelect from '../../../components/waterfall/WTreeSelect'
export default {
  name: 'ModelType',
  components: { WTreeSelect },
  props:{
    treeData: {
      type: Array,
      required: false,
      default: false
    }
  },
  data() {
    return {
      form: this.$form.createForm(this, { name: 'modelType' }),
      visible: false
    }
  },
  methods: {
    open() {
      this.visible = true
    }
  }
}
</script>

<style scoped>

</style>