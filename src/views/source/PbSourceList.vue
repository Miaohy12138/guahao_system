<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="医院编号">
              <a-input placeholder="请输入医院编号" v-model="queryParam.hospitalId"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="医生编号">
              <a-input placeholder="请输入医生编号" v-model="queryParam.doctorId"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="日期">
                <j-date placeholder="就诊日期" v-model="queryParam.time"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="就诊时间段">
                <a-input placeholder="请选择1、上午 2、下午 0 全天" v-model="queryParam.day"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
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
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('pb_source')">导出</a-button>
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
        :rowSelection="{fixed:true,selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"

        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="图片不存在" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定发布吗?" @confirm="() => push(record.id)">
                  <a>发布</a>
                </a-popconfirm>
              </a-menu-item>
               <a-menu-item>
                <a-popconfirm title="确定下架吗?" @confirm="() => xiajia(record.id)">
                  <a>下架</a>
                </a-popconfirm>
              </a-menu-item>
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

    <pbSource-modal ref="modalForm" @ok="modalFormOk"></pbSource-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PbSourceModal from './modules/PbSourceModal'
  import JDate from '@/components/jeecg/JDate.vue'
  import { httpAction } from '@/api/manage'
  export default {
    name: "PbSourceList",
    mixins:[JeecgListMixin],
    components: {
      JDate,
      PbSourceModal,
      httpAction
    },
    data () {
      return {
        description: 'pb_source管理页面',
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
          // {
          //   title:'createBy',
          //   align:"center",
          //   dataIndex: 'createBy'
          // },
          // {
          //   title:'createTime',
          //   align:"center",
          //   dataIndex: 'createTime',
          //   customRender:function (text) {
          //     return !text?"":(text.length>10?text.substr(0,10):text)
          //   }
          // },
          // {
          //   title:'updateBy',
          //   align:"center",
          //   dataIndex: 'updateBy'
          // },
          // {
          //   title:'updateTime',
          //   align:"center",
          //   dataIndex: 'updateTime',
          //   customRender:function (text) {
          //     return !text?"":(text.length>10?text.substr(0,10):text)
          //   }
          // },
          {
            title:'医院编号',
            align:"center",
            dataIndex: 'hospitalId'
          },
          {
            title:'医生编号',
            align:"center",
            dataIndex: 'doctorId'
          },
          {
            title:'价格',
            align:"center",
            dataIndex:'price'
          },
          {
            title:'日期',
            align:"center",
            dataIndex: 'time',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'就诊时间段',
            align:"center",
            dataIndex: 'day',
            customRender:function(t) {
              switch (t) {
                case '0':
                  return "全天"
                case '1':
                  return "上午"
                case '2':
                  return "下午"
              }
            }
          },
          {
            title:'状态',
            dataIndex:'status',
            align:"center",
            customRender:function(t) {
              switch (t) {
                case 0:
                  return "创建未发布"
                case 1:
                  return "已发布"
                case 2:
                  return "发布占用"
                case 3:
                  return "已下架"
              }
            }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/source/pbSource/list",
          delete: "/source/pbSource/delete",
          deleteBatch: "/source/pbSource/deleteBatch",
          exportXlsUrl: "/source/pbSource/exportXls",
          importExcelUrl: "source/pbSource/importExcel",
        },
        dictOptions:{},
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
      },
      push(id){
        let httpurl = "/source/pbSource/edit";
        let formData = {
          id:id,
          status:1
        };
        let method = 'post';
        httpAction(httpurl,formData,method).then((res)=>{
          this.reload()
        })
      },
      xiajia(id){
        let httpurl = "/source/pbSource/edit";
        let formData = {
          id:id,
          status:3
        };
        let method = 'post';
        httpAction(httpurl,formData,method).then((res)=>{
          this.reload()
        })
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>
