<template>
  <a-modal
    :title="title"
    :width="800"
    :ok=false
    :visible="visible"
    :confirmLoading="confirmLoading"
    :okButtonProps="{ props: {disabled: disableSubmit} }"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          style="display: none"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="menuId"
          :hidden="false"
          hasFeedback >
          <a-input id="menuId" placeholder="请输入menuId" v-decorator="['menuId', validatorRules.menuId ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="按钮名称"
          :hidden="false"
          hasFeedback >
          <a-input id="buttonName" placeholder="请输入按钮名称" v-decorator="['buttonName', validatorRules.buttonName ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="按钮编码">
          <a-input placeholder="请输入按钮编码" v-decorator="['buttonCode',validatorRules.buttonCode]" />
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { postAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import ATextarea from 'ant-design-vue/es/input/TextArea'
  export default {
    name: "SysDepartModal",
    components: { ATextarea },
    data () {
      return {
        departTree:[],
        orgTypeData:[],
        phoneWarning:'',
        departName:"",
        title:"操作",
        seen:false,
        visible: false,
        condition:true,
        disableSubmit:false,
        model: {},
        menuhidden:false,
        menuusing:true,
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        departName:{rules: [{ required: true, message: '请输入机构/部门名称!' }]},
        orgCode:{rules: [{ required: true, message: '请输入机构编码!' }]},
         mobile:{rules: [{validator:this.validateMobile}]}
        },
        url: {
          add: "/system/menuButton/add",
          update: "/system/menuButton/update",
        },
        dictDisabled:true,
      }
    },
    created () {
    },
    methods: {
      loadTreeData(){
        
      },
      add (depart) {
        console.log(depart)
        if(depart){
          this.seen = false;
          this.dictDisabled = false;
        }else{
          this.seen = true;
          this.dictDisabled = true;
        }
        this.edit(depart);
      },
      edit (record) {
          this.form.resetFields();
          this.model = Object.assign({}, record);
          this.visible = true;
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model,"menuId",'path','serviceId'))
          });
      },
      close () {
        this.$emit('close');
        this.disableSubmit = false;
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let formData = Object.assign(this.model, values);
            //时间格式化
            console.log(formData)
            var url = this.url.add
            if(this.model.id) {
              url = this.url.update
            }
            postAction(url,formData).then((res)=>{
              if(res.code === 200){
                that.$message.success(res.message);
                that.loadTreeData();
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
      validateMobile(rule,value,callback){
        if (!value || new RegExp(/^1([38][0-9]|4[579]|5[0-3,5-9]|6[6]|7[0135678]|9[89])\d{8}$/).test(value)){
          callback();
        }else{
          callback("您的手机号码格式不正确!");
        }

      }
    }
  }
</script>

<style scoped>

</style>