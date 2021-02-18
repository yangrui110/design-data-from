<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
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
                <a @click="handleDetail(record)">详情</a>
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

  </a-card>
</template>

<script>

  // import '@/assets/less/TableExpand.less'
  // import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction, postAction } from '@/api/manage';


  export default {
    name: "TenantList",
    mixins:[JeecgListMixin],
    components: {
    },
    data () {
      return {
        record: {},
        description: 'adad管理页面',
        // 表头
        columns: [
          {
            title:'角色名称',
            align:"center",
            dataIndex: 'roleName'
          },{
            title:'角色描述',
            align:"center",
            dataIndex: 'description'
          }
        ],
        url: {
          list: "/system/role/pageList",
          delete: "/system/user/delete",
          deleteBatch: "/system/user/batchDelete",
          userRoleList: '/system/user/getUserRoles',
          updateUserRole: "/system/userRole/updateUserRole"
        },
        dictOptions:{},
      }
    },
    created() {
    },
    computed: {
      importExcelUrl: function(){
        return '';
      },
    },
    methods: {
      initDictConfig(){
      },
      /**
       * 显示用户角色
       * @param record 当前的用户对象
      */
      show(record){
        // 1、查询该用户对应的角色信息
        var that = this
        this.record = record
        getAction(that.url.userRoleList,{userId: record.id}).then((resp)=>{
          if(resp.code === 200){
            // 更新选中的对象
            that.selectedRow = resp.data
            that.selectedRowKeys = resp.data.map(element => {
              return element.id
            });
            console.log(that.selectedRowKeys)
          }else {
            that.$message.error(resp.message)
          }
        })
      },
      submitForm(){
        var roleIds = ''
        for(var i=0;i<this.selectedRowKeys.length;i++){
          roleIds+=this.selectedRowKeys[i]+","
        }
        roleIds = roleIds.substring(0,roleIds.length-1)

// 
        var that = this
        postAction(that.url.updateUserRole,{roleIds: roleIds,userId: this.record.id}).then((resp)=>{
          if(resp.code === 200){
            this.$emit('ok')
            that.$message.info("修改成功")
          }else {
            that.$message.error("修改失败")
          }
        })

      }
    }
  }
</script>
<style scoped>
</style>