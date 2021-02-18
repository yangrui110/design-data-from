<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper" >
      <!-- 搜索区域 -->
      <a-form layout="inline" style="margin-left: 14px" >
        <a-row :gutter="10">
          <a-col :md="10" :sm="12">
            <a-form-item label="按钮名称" style="margin-left:8px">
              <a-input placeholder="请输入按钮名称" v-model="queryParam.buttonName"></a-input>
            </a-form-item>
            <a-form-item label="按钮编码" >
              <a-input placeholder="请输入按钮编码" v-model="queryParam.buttonCode"></a-input>
            </a-form-item>
          </a-col>
          <!--<a-col :md="8" :sm="8">-->
          <!--<a-form-item label="用户名称" :labelCol="{span: 5}" :wrapperCol="{span: 18, offset: 1}">-->
          <!--<a-input placeholder="请输入名称查询" v-model="queryParam.realname"></a-input>-->
          <!--</a-form-item>-->
          <!--</a-col>-->
          <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
            <a-col :md="6" :sm="24">
             <a-button type="primary" @click="searchQuery" icon="search" style="margin-left: 18px">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </a-col>
          </span>
        </a-row>
      </a-form>
    </div>
    <!-- 操作按钮区域 -->
    <div class="table-operator" :md="24" :sm="24" style="margin-top: -15px;margin-left: 14px" >
      <!--<a-button @click="handleEdit" type="primary" icon="edit" style="margin-top: 16px">用户编辑</a-button>-->
      <a-button @click="handleAddUserDepart" type="primary" icon="plus">新增</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            批量删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div style="margin-left: 14px" >
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
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
          <a @click="handleAddUserDepart(record)">编辑</a>
          <a @click="handleDelete(record)" style="margin-left: 5px">删除</a>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->
    <menu-button-modal ref="menuButtonModal" @ok="loadData"/>
  </a-card>
</template>

<script>
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import {getAction, postAction, deleteAction} from '@/api/manage'
  import MenuButtonModal from './MenuButtonModal'
  export default {
    name: "DeptUserInfo",
    mixins: [JeecgListMixin],
    components: {
      MenuButtonModal
    },
    data() {
     
        return {
        description: '用户信息',
        currentDeptId: '',
        // 表头
        columns: [{
            title: '按钮名称',
            align: "center",
            dataIndex: 'buttonName'
          },
          {
            title: '按钮编码',
            align: "center",
            dataIndex: 'buttonCode'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: {customRender: 'action'},
            align: "center",
            width: 150
          }],
        url: {
          list: "/system/menuButton/pageList",
          update: "/system/menuButton/update",
          delete: "/system/menuButton/delete",
          deleteBatch: "/system/menuButton/batchDelete",
        }
      }
    },
    created() {
    },

    methods: {
      searchReset() {
        this.queryParam = {}
        this.loadData(1);
      },
      loadData(arg) {
        if (!this.url.list) {
          this.$message.error("请设置url.list属性!")
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        //if (this.currentDeptId === '') return;
        let params = this.getQueryParams();//查询条件
        //params.depId = this.currentDeptId;
        params.data = Object.assign({menuId: this.currentDeptId}, params)
        postAction(this.url.list, params).then((res) => {
          if (res.code === 200 && res.data) {
            this.dataSource = res.data.data;
            this.ipagination.total = res.data.totalNum;
          }
        })
      },
      batchDel: function () {

        if (!this.url.deleteBatch) {
          this.$message.error("请设置url.deleteBatch属性!")
          return
        }
        if (!this.currentDeptId) {
          this.$message.error("未选中任何部门，无法取消部门与用户的关联!")
          return
        }

        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！');
          return;
        } else {
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            ids += this.selectedRowKeys[a] + ",";
          }
          var that = this;
          console.log(this.currentDeptId);
          this.$confirm({
            title: "确认取消",
            content: "是否删除该按钮?",
            onOk: function () {
              postAction(that.url.deleteBatch, that.selectionRows).then((res) => {
                if (res.code === 200) {
                  that.$message.success("删除按钮成功！");
                  that.loadData();
                  that.onClearSelected();
                } else {
                  that.$message.warning(res.message);
                }
              });
            }
          });
        }
      },
      handleDelete: function (record) {
        if (!this.url.delete) {
          this.$message.error("请设置url.delete属性!")
          return
        }
        if (!this.currentDeptId) {
          this.$message.error("未选中任何菜单，无法删除按钮!")
          return
        }

        var that = this;
        postAction(that.url.delete, record).then((res) => {
          if (res.code === 200) {
            that.$message.success("删除按钮成功！");
            if (this.selectedRowKeys.length>0){
               for(let i =0; i<this.selectedRowKeys.length;i++){
                   if (this.selectedRowKeys[i] == record.id){
                     this.selectedRowKeys.splice(i,1);
                     break;
                   }
               }
            }
            that.loadData();
          } else {
            that.$message.warning(res.message);
          }
        });
      },
      open(record) {
        //console.log(record);
        this.currentDeptId = record.id;
        this.loadData(1);
      },
      clearList() {
        this.currentDeptId = '';
        this.dataSource = [];
      },
      hasSelectDept() {
        if (this.currentDeptId == '') {
          this.$message.error("请选择一个部门!")
          return false;
        }
        return true;
      },
      handleAddUserDepart(record) {
        if (this.currentDeptId == '' ) {
          this.$message.error("请选择一个部门!")
        } else {
          var news = Object.assign(record, {menuId: this.currentDeptId})
          this.$refs.menuButtonModal.add(news)
          this.$refs.menuButtonModal.title = '新增'
        }
      },
      selectOK(data) {
        let params = {};
        params.depId = this.currentDeptId;
        params.userIdList = [];
        for (var a = 0; a < data.length; a++) {
          params.userIdList.push(data[a]);
        }
        console.log(params);
        postAction(this.url.edit, params).then((res) => {
          if (res.success) {
            this.$message.success(res.message);
            this.loadData();
          } else {
            this.$message.warning(res.message);
          }
        })
      }
    }
  }
</script>
<style scoped>
  /** Button按钮间距 */
  .ant-btn {
    margin-left: 3px
  }

  .ant-card {
    margin-left: -30px;
    margin-right: -30px;
  }

  .table-page-search-wrapper {
    margin-top: -16px;
    margin-bottom: 16px;
  }
</style>