<template>
  <div>
    <a-modal :visible="visible" :title="modalTitle" :width="600" @cancel="()=>{this.visible=false}" @ok="addModelLevel">
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
                v-decorator="['folderName', { rules: [{ required: true, message: '请选则模型类型' }] }]"
              />
            </a-form-item>
            <!-- <a-form-item label="模型类型">
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
            </a-form-item> -->
            <a-form-item label="描述">
              <a-input
                v-decorator="['remark', { rules: [{ required: false, message: '请输入描述' }] }]"
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
import { addModelFolder, updateModelFolder } from '../../../api/api'
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
      id:null,
      modalTitle:'新建模型层级',
      form: this.$form.createForm(this, { name: 'modelType' }),
      visible: false
    }
  },
  methods: {
    open(folderObj) {
      console.log(folderObj,'folderObj');
      this.visible = true
      this.form.resetFields()
      this.modalTitle=folderObj?'编辑模型层级':'新建模型层级'
      this.id = folderObj?folderObj.id:null
      if(folderObj){
        let {folderName,parentId,remark} = {...folderObj}
        // 编辑
        this.$nextTick(()=>{
          this.form.setFieldsValue({
            parentId:String(parentId),
            folderName:folderName,
            remark:remark
          });
        })
      }
      console.log(this.id,'id');
    },
    addModelLevel(){
      const that = this
      this.form.validateFields((err, values) => {
        if (!err) {
          // eslint-disable-next-line no-console
          console.log('Received values of form: ', values)
          if(this.id){
            let params = {
              ...values,
              id:this.id
            }
            // 更新文件夹
            updateModelFolder(params).then((res) =>{
              if (res.success) {
                that.$emit('refresh')
                that.$message.success('更新成功')
                that.visible = false
              } else {
                that.$message.error(res.message)
              }
            })
          }else{
            // 新增文件夹
            addModelFolder(values).then((res) =>{
              if (res.success) {
                that.$emit('refresh')
                that.$message.success('保存成功')
                that.visible = false
                // that.modal.visible = false
              } else {
                that.$message.error(res.message)
              }
            })
          }
        }
      })
    }
  }
}
</script>

<style scoped>

</style>