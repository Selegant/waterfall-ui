<template>
  <a-drawer
    :title="title"
    :maskClosable="true"
    :width="drawerWidth"
    placement="right"
    :closable="true"
    @close="handleCancel"
    :visible="visible"
    style="height: 100%;overflow: auto;padding-bottom: 53px;">

    <template slot="title">
      <div style="width: 100%;">
        <span>{{ title }}</span>
        <span style="display:inline-block;width:calc(100% - 51px);padding-right:10px;text-align: right">
          <a-button @click="toggleScreen" icon="appstore" style="height:20px;width:20px;border:0px"></a-button>
        </span>
      </div>

    </template>

    <a-spin :spinning="confirmLoading">
      <a-form-model ref="form" :model="model" :rules="validatorRules">

        <a-form-model-item label="连接名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="dataSourceName">
          <a-input placeholder="请输入连接名称" v-model="model.dataSourceName" />
        </a-form-model-item>

        <a-form-model-item label="数据库类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="dbType">
          <a-select placeholder="请选择数据库类型" v-model="model.dbType">
            <a-select-option v-for="(item,index) in dataSourceTypeOptions" :key="index" :value="item.value">
              {{item.name}}
            </a-select-option>
          </a-select>
        </a-form-model-item>

        <template>
          <a-form-model-item label="Host" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="host">
            <a-input placeholder="请输入数据库Host" v-model="model.host" />
          </a-form-model-item>
        </template>


        <template>
          <a-form-model-item label="Port" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="port">
            <a-input placeholder="请输入数据库端口号" v-model="model.port"  />
          </a-form-model-item>
        </template>

        <template>
          <a-form-model-item v-if="model.dbType==ORACLE" label="服务名" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="serverName">
            <a-input placeholder="请输入数据库服务名" v-model="model.serverName"  />
          </a-form-model-item>
        </template>

        <template>
          <a-form-model-item v-if="model.dbType==MYSQL||model.dbType==HIVE" label="Database" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="dbName">
            <a-input placeholder="请输入数据库名" v-model="model.dbName"  />
          </a-form-model-item>
        </template>


        <template>
          <a-form-model-item v-if="model.dbType==HIVE" label="defaultfs" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defaultfs">
            <a-input placeholder="请输入defaultFs" v-model="model.defaultfs"  />
          </a-form-model-item>
        </template>

        <template>
          <a-form-model-item v-if="model.dbType==HIVE" label="path" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="path">
            <a-input placeholder="请输入path" v-model="model.path"  />
          </a-form-model-item>
        </template>

        <template>
          <a-form-model-item v-if="model.dbType==HIVE" label="hadoopConfig" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="hadoopConfig">
            <a-textarea placeholder="请输入hadoopConfig" v-model="model.hadoopConfig"  :auto-size="{ minRows: 4, maxRows: 8 }" />
          </a-form-model-item>
        </template>

<!--        <template>-->
<!--          <a-form-model-item label="JDBC_URL" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="jdbcUrl">-->
<!--            <a-input placeholder="请输入数据库连接地址" v-model="model.jdbcUrl" @change="changeJdbcUrl" />-->
<!--          </a-form-model-item>-->
<!--        </template>-->

        <a-form-model-item label="用户名" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="username">
          <a-input placeholder="请输入用户名" v-model="model.username" />
        </a-form-model-item>

        <a-form-model-item label="密码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="password" >
          <a-input type="password" placeholder="请输入密码" v-model="model.password" />
        </a-form-model-item>

        <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input placeholder="请输入备注" v-model="model.remark" />
        </a-form-model-item>

<!--        <a-form-model-item label="数据库名" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="realname">-->
<!--          <a-input placeholder="数据库名" v-model="model.realname" />-->
<!--        </a-form-model-item>-->

      </a-form-model>
    </a-spin>


    <div class="drawer-bootom-button" v-show="!disableSubmit">
      <a-button @click="connection" type="danger"  :loading="confirmLoading" style="margin-right: .8rem;">测试连接</a-button>
      <a-popconfirm title="确定放弃编辑？" @confirm="handleCancel" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="confirmLoading">提交</a-button>
    </div>
  </a-drawer>
</template>

<script>
  import moment from 'moment'
  import Vue from 'vue'
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import { getAction } from '@/api/manage'
  import { addDataSource,testDataSource,getDataSourceTypeList,editDataSource,queryUserRole,queryall } from '@/api/api'
  import { disabledAuthFilter } from "@/utils/authFilter"
  import { duplicateCheck } from '@/api/api'

  const MYSQL='MYSQL'
  const ORACLE = 'ORACLE'
  const HIVE = 'HIVE'

  export default {
    name: "DataSourceModal",
    components: {
    },
    data () {
      return {
        MYSQL,
        ORACLE,
        HIVE,
        departDisabled: false, //是否是我的部门调用该页面
        roleDisabled: false, //是否是角色维护调用该页面
        modalWidth:800,
        drawerWidth:700,
        modaltoggleFlag:true,
        confirmDirty: false,
        userId:"", //保存用户id
        disableSubmit:false,
        dateFormat:"YYYY-MM-DD",
        validatorRules: {
          dataSourceName: [{ required: true, message: '请输入数据源名称!' },
            { validator: this.validateDataSourceName, }],
          dbType: [{ required: true, message: '请输入数据库类型!' }],
          jdbcUrl: [{ required: true, message: '请重新输入数据库地址!' }],
          host: [{ required: true, message: '请输入数据库Host!' }],
          port: [{ required: true, message: '请输入数据库端口号!' }],
          dbName: [{ required: true, message: '请输入数据库名!' }],
          serverName: [{ required: true, message: '请输入服务名!' }],
          username: [{ required: false, message: '请输入用户名!' }],
          password: [{ required: false, message: '请输入密码!' }],
          defaultfs: [{ required: true, message: '请输入defaultFs!' }],
          path: [{ required: true, message: '请输入path!' }],
          hadoopConfig: [{ required: true, message: '请输入hadoopConfig!' }, { validator: this.validateJSON }]
        },
        departIdShow:false,
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        uploadLoading:false,
        confirmLoading: false,
        headers:{},
        tenantsOptions: [],
        dataSourceTypeOptions:[],
        nextDepartOptions:[],
      }
    },
    mounted () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}
      this.initDataSourceTypeList()
    },
    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      add (params) {
        // console.log(params)
        // this.model.purpose = params.purpose
        // console.log(this.model.purpose)
        this.refresh();
        this.edit({activitiSync:'1',userIdentity:1,purpose:params.purpose});
      },
      edit (record) {
        let that = this;
        that.visible = true;
        //根据屏幕宽度自适应抽屉宽度
        this.resetScreenSize();
        that.userId = record.id;
        if(!record.jdbcUrl){
          record.jdbcUrl = ''
        }
        that.model = Object.assign({},{selectedroles:'',selecteddeparts:''}, record);
        // that.model.jdbcUrl = '12345'
        // //身份为上级显示负责部门，否则不显示
        // if(this.model.userIdentity==2){
        //   this.departIdShow=true;
        // }else{
        //   this.departIdShow=false;
        // }
        //
        // if(record.hasOwnProperty("id")){
        //   that.getUserRoles(record.id);
        //   that.getUserDeparts(record.id);
        // }
        // console.log('that.model=',that.model)
      },
      isDisabledAuth(code){
        return disabledAuthFilter(code);
      },
      //窗口最大化切换
      toggleScreen(){
        if(this.modaltoggleFlag){
          this.modalWidth = window.innerWidth;
        }else{
          this.modalWidth = 800;
        }
        this.modaltoggleFlag = !this.modaltoggleFlag;
      },
      // 根据屏幕变化,设置抽屉尺寸
      resetScreenSize(){
        let screenWidth = document.body.clientWidth;
        if(screenWidth < 500){
          this.drawerWidth = screenWidth;
        }else{
          this.drawerWidth = 700;
        }
      },
      //初始化数据库类型列表
      initDataSourceTypeList(){
        getDataSourceTypeList().then((res)=>{
          if(res.success){
            this.dataSourceTypeOptions = res.result.map((item,index,arr)=>{
              let c = {name:item.dbType, value:item.dbType}
              return c;
            })
            console.log('this.dataSourceTypeOptions: ',this.dataSourceTypeOptions)
          }
        });
      },
      changeType(e){
        const that = this
        this.dataSourceTypeOptions.forEach(function (item) {
          console.log(item.value)
          if(item.value === e){
            that.model.initUrl = item.url
            that.model.jdbcUrl = that.model.initUrl.concat(!that.model.host?'':that.model.host,':',!that.model.port?'':that.model.port,'/')
          }
        })
      },
      updateJdbcUrl(){
        const that = this
        if(!this.model.initUrl){
          this.dataSourceTypeOptions.forEach(function (item) {
            console.log(item.value)
            if(item.value === that.model.typeId){
              that.model.initUrl = item.url
              that.model.jdbcUrl = that.model.initUrl.concat(!that.model.host?'':that.model.host,':',!that.model.port?'':that.model.port,'/')
            }
          })
        }else {
          this.model.jdbcUrl = this.model.initUrl.concat(!this.model.host?'':this.model.host,':',!this.model.port?'':this.model.port,'/')
        }
      },
      changeJdbcUrl(e){
        // console.log(this.model.jdbcUrl.concat(e.data))
        // this.model.jdbcUrl = this.model.jdbcUrl.concat(e.data)
        console.log(e)
        console.log(this.model.jdbcUrl)
      },
      connection(){
        const that = this;
        console.log(that.model)
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            testDataSource(that.model).then((res)=>{
              that.confirmLoading = false;
              if(res.success){
                that.$message.success('连接成功');
                // console.log(res.success)
              }else {
                that.$message.error(res.message);
              }
            });
          }else {
            return false;
          }
        })
      },
      refresh () {
        this.userId=""
        this.nextDepartOptions=[];
        this.departIdShow=false;
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.disableSubmit = false;
        this.nextDepartOptions=[];
        this.departIdShow=false;
        this.$refs.form.resetFields();
      },
      moment,
      handleSubmit () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let obj;
            if(!this.model.id){
              obj=addDataSource(this.model);
            }else{
              obj=editDataSource(this.model);
            }
            obj.then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.error(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }else{
            return false;
          }
        })
      },
      handleCancel () {
        this.close()
      },
      validateDataSourceName(rule, value, callback) {
        var params = {
          tableName: 'waterfall_data_source',
          fieldName: 'data_source_name',
          fieldVal: value,
          dataId: this.userId
        }
        duplicateCheck(params).then((res) => {
          if (res.success) {
            callback()
          } else {
            callback('数据源名称已存在!')
          }
        })
      },
      validateJSON(rule, value, callback) {
        if (typeof value == 'string') {
          try {
            let obj = JSON.parse(value)
            if (typeof obj == 'object' && obj) {
              callback()
            } else {
              callback('hadoopConfig请输入JSON对象!')
            }
          } catch (e) {
            console.log('error：' + value + '!!!' + e)
            callback('hadoopConfig请输入JSON对象!')
          }
        }
      }
    }
  }
</script>

<style scoped>
  .avatar-uploader > .ant-upload {
    width:104px;
    height:104px;
  }
  .ant-upload-select-picture-card i {
    font-size: 49px;
    color: #999;
  }

  .ant-upload-select-picture-card .ant-upload-text {
    margin-top: 8px;
    color: #666;
  }

  .ant-table-tbody .ant-table-row td{
    padding-top:10px;
    padding-bottom:10px;
  }

  .drawer-bootom-button {
    position: absolute;
    bottom: -8px;
    width: 100%;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    left: 0;
    background: #fff;
    border-radius: 0 0 2px 2px;
  }
</style>