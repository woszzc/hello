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
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="证书名">
          <a-input placeholder="请输入证书名" v-decorator="['name', validatorRules.name ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="描述">
          <a-input placeholder="请输入描述" v-decorator="['description', {}]" />
        </a-form-item>
          <a-form-item label="轮播图" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-upload
                      listType="picture-card"
                      class="avatar-uploader"
                      :showUploadList="false"
                      :action="uploadAction"
                      :data="{'isup':1}"
                      :headers="headers"
                      :beforeUpload="beforeUpload"
                      @change="handleChange"
              >
                  <img v-if="picUrl" :src="getAvatarView()" alt="图片" style="height:104px;max-width:300px"/>
                  <div v-else>
                      <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
                      <div class="ant-upload-text">上传</div>
                  </div>
              </a-upload>
          </a-form-item>
          <a-form-item
                  :labelCol="labelCol"
                  :wrapperCol="wrapperCol"
                  label="排序">
              <a-input-number v-decorator="[ 'sort', {initialValue:5}]" />
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
  import Vue from 'vue'
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "CredentialModal",
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
        name:{rules: [{ required: true, message: '请输入证书名!' }]},
        },
        url: {
          add: "/person/credential/add",
          edit: "/person/credential/edit",
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
        },
        headers:{},
        picUrl: "",
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}
    },

    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.picUrl = "afa";
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'name','description','img','sort','status'))
		  //时间格式化
        });

      },

      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        this.picUrl = "";
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if(response.success){
            this.model.img = response.message;
            this.picUrl = "Has no pic url yet";
          }else{
            this.$message.warning(response.message);
          }
        }
      },
      getAvatarView(){
        return this.url.imgerver +"/"+ this.model.img;
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