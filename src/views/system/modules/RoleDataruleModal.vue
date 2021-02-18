<template>
  <a-drawer
    title="数据规则/按钮权限配置"
    width="365"
    :closable="false"
    @close="onClose"
    :visible="visible"
  >

    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="按钮配置" key="1">

        <a-checkbox-group v-model="dataruleChecked" v-if="dataruleList.length>0">
          <a-row>
            <a-col :span="24" v-for="(item,index) in dataruleList" :key=" 'dr'+index ">
              <a-checkbox :value="item.id">{{ item.buttonName }}--{{item.buttonCode}}</a-checkbox>
            </a-col>

            <a-col :span="24">
              <div style="width: 100%;margin-top: 15px">
                <a-button @click="saveDataruleForRole(1)" type="primary" size="small" icon="save">点击保存</a-button>
              </div>
            </a-col>
          </a-row>
        </a-checkbox-group>
        <div v-else><h3>无配置信息!</h3></div>

      </a-tab-pane>
      <a-tab-pane tab="接口配置" key="2">

        <a-checkbox-group v-model="apiChecked" v-if="apiList.length>0">
          <a-row>
            <a-col :span="24" v-for="(item,index) in apiList" :key=" 'dr'+index ">
              <a-checkbox :value="item.id">{{ item.path }}</a-checkbox>
            </a-col>

            <a-col :span="24">
              <div style="width: 100%;margin-top: 15px">
                <a-button @click="saveDataruleForRole(2)" type="primary" size="small" icon="save">点击保存</a-button>
              </div>
            </a-col>
          </a-row>
        </a-checkbox-group>
        <div v-else><h3>无配置信息!</h3></div>

      </a-tab-pane>
    </a-tabs>

  </a-drawer>
</template>

<script>
  import ARow from 'ant-design-vue/es/grid/Row'
  import ACol from 'ant-design-vue/es/grid/Col'
  import { getAction,postAction } from '@/api/manage'

  export default {
    name: 'RoleDataruleModal',
    components: { ACol, ARow },
    data(){
      return {
        functionId:'',
        roleId:'',
        visible:false,
        tabList: [{
          key: '1',
          tab: '数据规则',
        }, {
          key: '2',
          tab: '按钮权限',
        }],
        activeTabKey: '1',
        url:{
          datarule:"/system/menuButton/listButtons",
          apiList:"/system/menuApi/listApis",
          saveRoleButton: '/system/rolePermission/save'
        },
        dataruleList:[],
        dataruleChecked:[],
        apiList: [],
        apiChecked: []
      }
    },
    methods:{
      loadData(){
        var that = this
        /**
         * 获取到按钮配置
         * 
        */
        getAction(that.url.datarule,{roleId: that.roleId,menuId: that.functionId}).then(res=>{
          console.log(res)
          if(res.code === 200){
            that.dataruleList = res.data.menuButtons
            that.dataruleChecked = res.data.roleButtons.map(item=> item.id)
            
          }
        })
        /**
         * 获取到接口配置
         * 
        */
        getAction(that.url.apiList,{roleId: that.roleId,menuId: that.functionId}).then(res=>{
          console.log(res)
          if(res.code === 200){
            that.apiList = res.data.menuApis
            that.apiChecked = res.data.roleApis.map(item=> item.id)
            
          }
        })
      },
      saveDataruleForRole(type){
        var dataChecked = this.dataruleChecked
        if(type===2){
          dataChecked = this.apiChecked
        }
        if(!dataChecked || dataChecked.length==0){
          this.$message.warning("请注意，现未勾选任何数据权限!")
        }
        let params = {
          roleId:this.roleId,
          permissionIds:dataChecked.join(","),
          type: type
        }
        console.log("保存数据权限",params)
        postAction(this.url.saveRoleButton,params).then(res=>{
          if(res.code === 200){
            this.$message.success(res.message)
          }else{
            this.$message.error(res.message)
          }
        })
      },
      show(functionId,roleId){
        this.onReset()
        this.functionId = functionId
        this.roleId = roleId
        this.visible=true
        this.loadData()
      },
      onClose(){
        this.visible=false
        this.onReset()
      },
      onTabChange (key) {
        this.activeTabKey = key
      },
      onReset(){
        this.functionId=''
        this.roleId=''
        this.dataruleList=[]
        this.dataruleChecked=[]
      }
    }
  }
</script>

<style scoped>

</style>