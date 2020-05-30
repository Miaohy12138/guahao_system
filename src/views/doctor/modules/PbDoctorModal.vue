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

        <a-form-item label="姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'name', validatorRules.name]" placeholder="请输入姓名"></a-input>
        </a-form-item>
        <a-form-item label="医生等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-search-select-tag v-decorator="['level', validatorRules.level]" dict="doctor_level" />
        </a-form-item>

        <a-form-item label="hospitalId" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-multi-select-tag type="list_multi"
                              v-model="hospitalId"
                              :options="hospitalIds"
                              v-decorator="['hospitalId', validatorRules.hospitalId]"
                              :trigger-change="true"
                              placeholder="请选择hospitalId"/>
        </a-form-item>
        <a-form-item label="医院id" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-multi-select-tag
                  type="list_multi"
                  v-model="departmentId"
                  :options="departmentIds"
                  v-decorator="['departmentId', validatorRules.departmentId]"
                  :trigger-change="true"
                  placeholder="请选择医院id"/>
        </a-form-item>

        <a-form-item label="擅长疾病" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'goodDisease', validatorRules.goodDisease]" placeholder="请输入擅长疾病"></a-input>
        </a-form-item>
        <a-form-item label="introduction" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'introduction', validatorRules.introduction]" placeholder="请输入introduction"></a-input>
        </a-form-item>
        <a-form-item label="医生头像" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['imageUrl', validatorRules.imageUrl]" :trigger-change="true"></j-upload>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JMultiSelectTag from "@/components/dict/JMultiSelectTag"
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'
  import JUpload from '@/components/jeecg/JUpload'
  export default {
    name: "PbDoctorModal",
    components: {
      JMultiSelectTag,
      JSearchSelectTag,
      JUpload
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
          imageUrl:{rules: [
            ]},
          name: {rules: [
          ]},
          level: {rules: [
          ]},
          hospitalId: {rules: [
          ]},
          departmentId: {rules: [
          ]},
          goodDisease: {rules: [
          ]},
          introduction: {rules: [
          ]},
        },
        url: {
          add: "/doctor/pbDoctor/add",
          edit: "/doctor/pbDoctor/edit",
        },
        hospitalId: '',
        hospitalIds: [],
        departmentId: '',
        departmentIds: [],
        tempIds:[]
      }
    },
    created () {
      let httpurl = '/hospital/hospital/listNoPage'
      let formData = {}
      let method = 'post'
      httpAction(httpurl, formData, method).then((res) => {
        let result = res.result
        console.log(result)
        for (let i = 0; i < result.length; i++) {
          this.hospitalIds.push({ label: result[i].name, value: '' + result[i].id })
        }
      })

      let httpurl2 = '/department/pbDepartment/list'
      let formData2 = {
      }

      let method2 = 'post'
      httpAction(httpurl2, formData2, method2).then(res=>{
        let result = res.result;
        this.departmentIds.splice(0, this.departmentIds.length);
        for (let i = 0; i < result.length; i++) {
          this.departmentIds.push({ label: result[i].name, value: '' + result[i].id })
        }
        this.tempIds = this.departmentIds;
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
          this.form.setFieldsValue(pick(this.model,'name','level','hospitalId','departmentId','goodDisease','introduction','createTime','updateTime','imageUrl'))
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
        this.form.setFieldsValue(pick(row,'name','level','hospitalId','departmentId','goodDisease','introduction','createTime','updateTime','imageUrl'))
      },
    },
    watch:{
      // hospitalId(newId,oldId){
      //   let httpurl = '/hospital/hospital/queryById?id='+newId
      //   let formData = {}
      //   let method = 'get'
      //   httpAction(httpurl, formData, method).then((res) => {
      //     let result = res.result.departments
      //     let arr = result.split(",");
      //     this.departmentId = []
      //     for(let i = 0;i<arr.length;i++){
      //       for(let j = 0;j<this.tempIds.length;j++){
      //         if(this.tempIds[j].value==arr[i]){
      //           this.departmentIds.push({ label: this.tempIds[j].label, value: '' + this.tempIds[j].value })
      //         }
      //       }
      //     }
      //
      //   })
      //
      // }
    }
  }
</script>
