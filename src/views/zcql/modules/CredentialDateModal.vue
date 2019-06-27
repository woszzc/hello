<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

          <a-form-item label="证书" :labelCol="labelCol" :wrapperCol="{xs: { span: 24 },sm: { span: 6 },}">
              <a-select  v-decorator="[ 'credentialId', {}]" placeholder="">
                  <a-select-option v-for="(item,index) in optionCredential" :key="index" :value="item.id" >{{item.name}}</a-select-option>
              </a-select>
          </a-form-item>

         <!-- <a-form-item label="地址" :labelCol="labelCol" :wrapperCol="{xs: { span: 24 },sm: { span: 9 },}">
              <a-cascader :options="options" @change="onChange" placeholder="Please select" />
          </a-form-item>-->



          <a-form-item
                  :labelCol="labelCol"
                  :wrapperCol="wrapperCol"
                  label="考试地址">
              <!--<a-input placeholder="请输入角色编码" :disabled="roleDisabled" v-decorator="[ 'address', {}]" />-->
              <a-textarea :rows="5" placeholder="..." v-decorator="[ 'address', {} ]" />
              <!--<a-textarea :rows="5" placeholder="..." v-decorator="[ 'address', {} ]" />-->
          </a-form-item>


        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="考试开始时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'examDateStart', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="考试截止时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'examDateEnd', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="截止报名时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'applyDateEnd', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="{xs: { span: 24 },sm: { span: 6 },}"
          label="费用">

          <a-input-number v-decorator="[ 'money', {}]" />
            <span class="money"></span>
        </a-form-item>
          <a-form-item label="状态" :labelCol="labelCol" :wrapperCol="{xs: { span: 24 },sm: { span: 3 },}">
              <a-select  v-decorator="[ 'status', {initialValue:1}]" placeholder="">
                  <a-select-option :value="1" >正常</a-select-option>
                  <a-select-option :value="0">关闭</a-select-option>
              </a-select>
          </a-form-item>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import { getAction } from '../../../api/manage'

  export default {
    name: "CredentialDateModal",
    data () {
      return {
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

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        credentialId:{rules: [{ required: true, message: '请输入证书!' }]},
        },
        url: {
          add: "/person/credentialDate/add",
          edit: "/person/credentialDate/edit",
        },
        optionCredential: [],
        options: [],
      }
    },
    created () {

    },
    methods: {
      onChange(value,text) {
        console.log(value);
        console.log(value[1]);
        console.log(text[0].label)
      },

      //获取证书select
      getCreadential() {
        getAction("/person/credential/list").then((res) =>{
          if(res.success){
            this.optionCredential = res.result.records;
          }else{
            this.$message.error(res.message);
          }

        })
      },

      getAddress() {
        getAction("/api/json/area").then((res) =>{
          this.options = res;
        })
      },

      add () {
        this.getAddress();
        this.getCreadential();
        this.edit({});
      },
      edit (record) {
        this.getAddress();
        this.getCreadential();
        this.form.resetFields();
        //this.model.address = record.address;
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'credentialId','address','money','status'))
		  //时间格式化
          this.form.setFieldsValue({examDateStart:this.model.examDateStart?moment(this.model.examDateStart):null})
          this.form.setFieldsValue({examDateEnd:this.model.examDateEnd?moment(this.model.examDateEnd):null})
          this.form.setFieldsValue({applyDateEnd:this.model.applyDateEnd?moment(this.model.applyDateEnd):null})
        });

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
            //时间格式化
            formData.examDateStart = formData.examDateStart?formData.examDateStart.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.examDateEnd = formData.examDateEnd?formData.examDateEnd.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.applyDateEnd = formData.applyDateEnd?formData.applyDateEnd.format('YYYY-MM-DD HH:mm:ss'):null;
            
            console.log(formData)
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


    }
  }
</script>

<style lang="less" scoped>

</style>