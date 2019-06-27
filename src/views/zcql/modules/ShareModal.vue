<template>
  <a-modal
    :title="title"
    :width="950"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    okText="审核通过"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"

          label="共享人">
          <a-input placeholder="请输入共享人" disabled v-decorator="['userId', validatorRules.userId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="标题">
          <a-textarea placeholder="请输入标题" disabled v-decorator="['title', validatorRules.title ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="共享类型">
          <a-input-number v-decorator="[ 'shareType', validatorRules.shareType ]" disabled />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="共享内容类别">
          <a-input-number v-decorator="[ 'contentType', validatorRules.contentType ]" disabled />
        </a-form-item>
       <!-- <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="共享内容">
          <a-input placeholder="请输入共享内容" v-decorator="['content', validatorRules.content ]" disabled />
        </a-form-item>-->

          <!-- JEditor -->
          <a-row :gutter="24">
              <a-col :span="24">
                  <a-form-item label="共享内容" style="min-height: 300px">
                      <j-editor v-model="jeditor.value"/>
                  </a-form-item>
              </a-col>
          </a-row>

       <!-- <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="附件">
          <a-input placeholder="请输入附件" v-decorator="['annex', {}]" disabled />
        </a-form-item>-->

          <a-form-item
                  :labelCol="labelCol"
                  :wrapperCol="wrapperCol"
                  label="附件">
          <a-upload action="https://www.mocky.io/v2/5cc8019d300000980a055e76" :defaultFileList="defaultFileList" >
              <!--<a-button>
                  <a-icon type="upload" /> Upload
              </a-button>-->
          </a-upload>
          </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="共享时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'shareTime', {}]" disabled />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="审核状态">
          <a-input-number v-decorator="[ 'checkStatus', {}]" disabled />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="审核通过时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'checkTime', {}]" disabled />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="价格">
          <a-input-number v-decorator="[ 'price', {}]" disabled />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="共享状态">
          <a-input-number v-decorator="[ 'shareStatus', {}]" disabled />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="修改时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'shareUpdateTime', {}]" disabled />
        </a-form-item>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import JEditor from '@/components/jeecg/JEditor'

  export default {
    name: "ShareModal",
    components: {
      JEditor,
    },
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
        userId:{rules: [{ required: true, message: '请输入共享人!' }]},
        title:{rules: [{ required: true, message: '请输入标题!' }]},
        shareType:{rules: [{ required: true, message: '请输入共享类型!' }]},
        contentType:{rules: [{ required: true, message: '请输入共享内容类别!' }]},
        content:{rules: [{ required: true, message: '请输入共享内容!' }]},
        },
        url: {
          add: "/person/share/add",
          edit: "/person/share/edit",
        },

        defaultFileList: [{
          uid: '1',
          name: 'baidu.png',
          status: 'done',
          response: 'Server Error 500', // custom error message to show
          url: 'https://dl.softmgr.qq.com/original/im/QQ9.1.3.25332.exe',
        }, {
          uid: '2',
          name: 'yyy.png',
          status: 'done',
          url: 'http://www.baidu.com/yyy.png',
        }],
        jeditor: {
          value: '',
        },
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        console.log(record.content)
        this.jeditor.value = record.content;
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'userId','title','shareType','contentType','content','annex','checkStatus','price','shareStatus'))
		  //时间格式化
          this.form.setFieldsValue({shareTime:this.model.shareTime?moment(this.model.shareTime):null})
          this.form.setFieldsValue({checkTime:this.model.checkTime?moment(this.model.checkTime):null})
          this.form.setFieldsValue({shareUpdateTime:this.model.shareUpdateTime?moment(this.model.shareUpdateTime):null})
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
            formData.shareTime = formData.shareTime?formData.shareTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.checkTime = formData.checkTime?formData.checkTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.shareUpdateTime = formData.shareUpdateTime?formData.shareUpdateTime.format('YYYY-MM-DD HH:mm:ss'):null;
            
            console.log(formData)
            //pass
            httpAction("/person/share/pass",formData,"post").then((res)=>{
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