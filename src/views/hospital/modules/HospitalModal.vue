<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="院名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入院名"></a-input>
        </a-form-item>
        <a-form-item label="医院地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'address', validatorRules.address]" placeholder="请输入医院地址"></a-input>
        </a-form-item>
        <a-form-item label="医院号码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'telephone', validatorRules.telephone]" placeholder="请输入医院号码"></a-input>
        </a-form-item>
        <a-form-item label="医院介绍" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'introduction', validatorRules.introduction]" placeholder="请输入医院介绍"></a-input>
        </a-form-item>
        <a-form-item label="医院等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-search-select-tag v-decorator="['level', validatorRules.level]" dict="hospital_level" />
        </a-form-item>
        <a-form-item label="医院图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['coverUrl', validatorRules.coverUrl]" :trigger-change="true"></j-upload>
        </a-form-item>
        <a-form-item label="科室列表" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-multi-select-tag
                  v-model="selectValue"
                  :options="dictOptions"
                   v-decorator="['departments', validatorRules.departments]"
                  :trigger-change="true"
                  dictCode=""
                  placeholder="请选择科室列表"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JUpload from '@/components/jeecg/JUpload'
  import JMultiSelectTag from "@/components/dict/JMultiSelectTag"
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'

  export default {
    name: "HospitalModal",
    components: {
      JUpload,
      JMultiSelectTag,
      JSearchSelectTag,
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
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
        confirmLoading: false,
        validatorRules: {
          name: {rules: [
            {required: true, message: '请输入院名!'},
          ]},
          address: {rules: [
          ]},
          telephone: {rules: [
            {required: true, message: '请输入医院号码!'},
          ]},
          introduction: {rules: [
          ]},
          level: {rules: [
            {required: true, message: '请输入医院等级!'},
          ]},
          coverUrl: {rules: [
            {required: true, message: '请输入医院图片!'},
          ]},
          departments: {rules: [
              {required: true, message: '请选择科室'},
          ]},
        },
        url: {
          add: "/hospital/hospital/add",
          edit: "/hospital/hospital/edit",
        },
        selectValue:"",
        checkboxValue:"",
        dictOptions:[]
      }
    },
    created () {
      let httpurl = "/department/pbDepartment/list";
      let formData = {}
      let method = 'post';

       httpAction(httpurl,formData,method).then((res)=>{
         let result = res.result;
         for(let i= 0;i<result.length;i++){
           this.dictOptions.push({label:result[i].name,value:''+result[i].id})
         }
       })
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'name','address','telephone','introduction','level','coverUrl','createTime','updateTime','departments'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }

        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'name','address','telephone','introduction','level','coverUrl','createTime','updateTime','departments'))
      },


    }
  }
</script>
