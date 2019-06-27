<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

            <a-col :md="6" :sm="8">
                <a-form-item label="证书">
                    <a-select v-model="queryParam.credentialId" @change="selectChange" placeholder="请选择证书查询">
                        <a-select-option v-for="(item,index) in optionCredential" :key="index" :value="item.id" >{{item.name}}</a-select-option>
                    </a-select>
                </a-form-item>
            </a-col>

          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('考证时间安排')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <credentialDate-modal ref="modalForm" @ok="modalFormOk"></credentialDate-modal>
  </a-card>
</template>

<script>
  import CredentialDateModal from './modules/CredentialDateModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import {httpAction} from '../../api/manage'
  import {getAction} from '../../api/manage'
  import {initDictOptions, filterDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: "CredentialDateList",
    mixins:[JeecgListMixin],
    components: {
      CredentialDateModal
    },
    data () {
      return {
        description: '考证时间安排管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
           },
		   {
            title: '证书',
            align:"center",
            dataIndex: 'credentialId',
             customRender: (text, record, index) => {
               return filterDictText(this.credentialDict, text);
             }
           },
          {
            title: '考试地点',
            align:"center",
            dataIndex: 'address'
          },
		   {
            title: '考试开始时间',
            align:"center",
            dataIndex: 'examDateStart'
           },
		   {
            title: '考试截止时间',
            align:"center",
            dataIndex: 'examDateEnd'
           },
		   {
            title: '截止报名时间',
            align:"center",
            dataIndex: 'applyDateEnd'
           },
		   {
            title: '费用',
            align:"center",
            dataIndex: 'money'
           },
		   {
            title: '状态',
            align:"center",
            dataIndex: 'status',
             customRender: (text, record, index) => {
               return filterDictText(this.status, text);
             }
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/person/credentialDate/list",
          delete: "/person/credentialDate/delete",
          deleteBatch: "/person/credentialDate/deleteBatch",
          exportXlsUrl: "person/credentialDate/exportXls",
          importExcelUrl: "person/credentialDate/importExcel",
       },
        optionCredential: [],
        status: [],
        credentialDict: [],
    }
  },

  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    }
  },
    created() {
      this.getCreadential();
      this.initDictConfig();

    },
    methods: {
      //获取证书select
      getCreadential() {
        getAction("/person/credential/getCredentialOption").then((res) =>{
          if(res.success){
            this.optionCredential = res.result;
            //console.log(res.result);
          }else{
            this.$message.error(res.message);
          }

        })
      },
      //自己的字典
      getCreadential() {
        getAction("/person/credential/getCredentialDict").then((res) =>{
          if(res.success){
            this.credentialDict = res.result;
            //console.log(res.result);
          }else{
            this.$message.error(res.message);
          }

        })
      },

      //下拉选改变
      selectChange(value) {
        this.searchQuery();
      },



      initDictConfig() {
        //初始化字典 - 性别
        initDictOptions('valid_status').then((res) => {
          if (res.success) {
            this.status = res.result;
          }
        });
      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>