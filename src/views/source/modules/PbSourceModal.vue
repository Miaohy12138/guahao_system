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

        <a-form-item label="医院" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-multi-select-tag
                  v-model="hospitalId"
                  :options="hospitalIds"
                  v-decorator="['hospitalId', validatorRules.hospitalId]"
                  :trigger-change="true"
                  placeholder="医院id"/>
        </a-form-item>
        <a-form-item label="科室" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-multi-select-tag
                  v-model="departmentId"
                  :options="departmentIds"
                  v-decorator="['departmentId', validatorRules.departmentId]"
                  :trigger-change="true"
                  placeholder="科室选择"/>
        </a-form-item>
        <a-form-item label="医生" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-multi-select-tag
                  v-model="doctorId"
                  :options="doctorIds"
                  v-decorator="['doctorId', validatorRules.doctorId]"
                  :trigger-change="true"
                  placeholder="医生id"/>
        </a-form-item>
        <a-form-item label="日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择日期" v-decorator="[ 'time', validatorRules.time]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="就诊时间段" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag v-decorator="[ 'day', validatorRules.day]" placeholder="请选择就诊时间段" dictCode="c_time" :triggerChange="true"/>
        </a-form-item>
        <a-form-item label="价格" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'price', validatorRules.price]" placeholder="请输入价格"></a-input>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'
  import JSearchSelectTag from '@/components/dict/JSearchSelectTag'
  import JMultiSelectTag from "@/components/dict/JMultiSelectTag"
  import JDictSelectTag from '../../../components/dict/JDictSelectTag'
  export default {
    name: "PbSourceModal",
    components: {
      JDate,
      JSearchSelectTag,
      JMultiSelectTag,
      JDictSelectTag
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
          hospitalId: {rules: []},
          doctorId: {rules: []},
          departmentId: {rules: []},
          time: {rules: []},
          day: {rules: []},
          price:{rules:[]}
        },
        url: {
          add: "/source/pbSource/add",
          edit: "/source/pbSource/edit",
        },
        selectValue:"",
        hospitalIds:[],
        doctorIds:[],
        hospitalId:'',
        doctorId:'',
        departmentIds:[],
        departmentId:'',
      }
    },
    created () {

      let httpurl = "/hospital/hospital/listNoPage";
      let formData = {}
      let method = 'post';
      httpAction(httpurl,formData,method).then((res)=>{
        let result = res.result;
        for(let i= 0;i<result.length;i++){
          this.hospitalIds.push({label:result[i].name,value:''+result[i].id})
        }
      });

      let httpurl2 = "/doctor/pbDoctor/listByHId?hid="+this.hospitalId;
      let formData2 = {}
      let method2 = 'get';
      httpAction(httpurl2,formData2,method2).then((res)=>{
        let result = res.result;
        this.doctorIds = []
        for(let i= 0;i<result.length;i++){
          this.doctorIds.push({label:result[i].name,value:''+result[i].id})
        }
      });

      let httpurl3 = "/department/pbDepartment/listByDid";
      let formData3 = {
        id:this.doctorId
      };
      let method3  = 'post';
      httpAction(httpurl3,formData3,method3).then(res =>{
        let result = res.result;
        this.departmentIds = [{label:result.name,value:''+result.id}]
      });

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
          this.form.setFieldsValue(pick(this.model,'createBy','createTime','updateBy','updateTime','hospitalId','departmentId','doctorId','time','day','price'))
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
        this.form.setFieldsValue(pick(row,'createBy','createTime','updateBy','updateTime','hospitalId','departmentId','doctorId','time','day','price'))
      },


    },
    watch:{
      hospitalId(n,o){
        let httpurl = "/department/pbDepartment/listByHid";
        let formData = {
          id:n
        };
        let method = 'post';
        httpAction(httpurl,formData,method).then((res)=>{
          let result = res.result;
          this.departmentIds = []
          for(let i= 0;i<result.length;i++){
            this.departmentIds.push({label:result[i].name,value:''+result[i].id})
          }
        })
      },
      departmentId(n,o){
        let httpurl = "/doctor/pbDoctor/listByDId?did="+n+"&hid="+this.hospitalId;
        let method  = 'get';
        httpAction(httpurl,{},method).then(res =>{
          let result = res.result;
          for(let i= 0;i<result.length;i++){
            this.doctorIds.push({label:result[i].name,value:''+result[i].id})
          }

        })
      }
    }
  }
</script>
