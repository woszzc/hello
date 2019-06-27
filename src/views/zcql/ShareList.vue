<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="共享人">
              <a-input placeholder="请输入共享人" v-model="queryParam.userId"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="标题">
              <a-input placeholder="请输入标题" v-model="queryParam.title"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="共享类型">
              <a-input placeholder="请输入共享类型" v-model="queryParam.shareType"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="共享内容类别">
              <a-input placeholder="请输入共享内容类别" v-model="queryParam.contentType"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="共享内容">
              <a-input placeholder="请输入共享内容" v-model="queryParam.content"></a-input>
            </a-form-item>
          </a-col>
        </template>
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
      <a-button @click="handlePass" type="primary" >审核</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('内容审核')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">

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
          <a @click="handleEdit(record)">详情</a>

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
    <share-modal ref="modalForm" @ok="modalFormOk"></share-modal>
  </a-card>
</template>

<script>
  import ShareModal from './modules/ShareModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "ShareList",
    mixins:[JeecgListMixin],
    components: {
      ShareModal
    },
    data () {
      return {
        description: '内容审核管理页面',
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
            title: '共享人',
            align:"center",
            dataIndex: 'userId'
           },
		   {
            title: '标题',
            align:"center",
            dataIndex: 'title'
           },
		   {
            title: '共享类型',
            align:"center",
            dataIndex: 'shareType'
           },
		   {
            title: '共享内容类别',
            align:"center",
            dataIndex: 'contentType'
           },
		   {
            title: '共享内容',
            align:"center",
            dataIndex: 'content'
           },
		   {
            title: '附件',
            align:"center",
            dataIndex: 'annex'
           },
		   {
            title: '共享时间',
            align:"center",
            dataIndex: 'shareTime'
           },
		   {
            title: '审核状态',
            align:"center",
            dataIndex: 'checkStatus'
           },
		   {
            title: '审核通过时间',
            align:"center",
            dataIndex: 'checkTime'
           },
		   {
            title: '价格',
            align:"center",
            dataIndex: 'price'
           },
		   {
            title: '共享状态',
            align:"center",
            dataIndex: 'shareStatus'
           },
		   {
            title: '修改时间',
            align:"center",
            dataIndex: 'shareUpdateTime'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/person/share/list",
          delete: "/person/share/delete",
          deleteBatch: "/person/share/deleteBatch",
          exportXlsUrl: "person/share/exportXls",
          importExcelUrl: "person/share/importExcel",
       },
        isorter:{
          column: '',
          order: '',
        },

    }
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    }
  },
    methods: {
      handlePass(){
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！');
          return;
        }else {
          console.log("审核......")
          console.log(this.selectedRowKeys);
        }
      },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>